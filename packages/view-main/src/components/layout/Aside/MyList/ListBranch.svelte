<!--
  新增文件：packages/view-main/src/components/layout/Aside/MyList/ListBranch.svelte
  作用：渲染一个列表项 + 它按 parentId 挂在下面的子列表（可递归、可折叠）。
-->
<script lang="ts">
  import ListBranch from './ListBranch.svelte'
  import ListItem from './ListItem.svelte'
  import { useUserList } from '@/modules/musicLibrary/reactive.svelte'

  interface Props {
    listInfo: AnyListen.List.MyListInfo
    index: number
    activeId: string | null
    picStyle: string
    onmenu: (item: AnyListen.List.MyListInfo, event: MouseEvent) => void
  }
  let { listInfo, index, activeId, picStyle, onmenu }: Props = $props()

  const STORAGE_KEY = 'sidebar-child-list-expanded'
  const readExpanded = (id: string) => {
    try {
      return (JSON.parse(localStorage.getItem(STORAGE_KEY) ?? '{}') as Record<string, boolean>)[id] === true
    } catch {
      return false
    }
  }
  const writeExpanded = (id: string, val: boolean) => {
    try {
      const map = JSON.parse(localStorage.getItem(STORAGE_KEY) ?? '{}') as Record<string, boolean>
      if (val) map[id] = true
      else delete map[id]
      localStorage.setItem(STORAGE_KEY, JSON.stringify(map))
    } catch {}
  }

  // 默认列表（试听列表 / 喜欢列表等）没有子列表，不查询
  const children = listInfo.type == 'default' ? null : useUserList(listInfo.id)
  const childLists = $derived(children?.val ?? [])
  let expanded = $state(readExpanded(listInfo.id))

  const toggle = (event: MouseEvent) => {
    event.stopPropagation()
    expanded = !expanded
    writeExpanded(listInfo.id, expanded)
  }
</script>

<div class="branch">
  <div class="row">
    <ListItem
      {listInfo}
      active={activeId == listInfo.id}
      {index}
      {picStyle}
      oncontextmenu={(event: MouseEvent) => {
        event.preventDefault()
        event.stopPropagation()
        onmenu(listInfo, event)
      }}
    />
    {#if childLists.length}
      <button
        class="toggle"
        class:expanded
        type="button"
        aria-label={expanded ? 'Collapse' : 'Expand'}
        aria-expanded={expanded}
        onclick={toggle}
      ></button>
    {/if}
  </div>
  {#if expanded && childLists.length}
    <ul class="children">
      {#each childLists as child (child.id)}
        <!-- 不加 draggable-item / data-id，避免被顶层 sortable 当成可排序项 -->
        <li class="child-item">
          <ListBranch listInfo={child} {index} {activeId} {picStyle} {onmenu} />
        </li>
      {/each}
    </ul>
  {/if}
</div>

<style lang="less">
  .row {
    position: relative;
  }
  .children {
    padding-left: 14px;
  }
  .toggle {
    position: absolute;
    top: 50%;
    right: 6px;
    width: 18px;
    height: 18px;
    padding: 0;
    border: none;
    background: transparent;
    cursor: pointer;
    transform: translateY(-50%);
    opacity: 0.6;
    &:hover {
      opacity: 1;
    }
    &::before {
      content: '';
      position: absolute;
      top: 50%;
      left: 50%;
      border-style: solid;
      border-width: 4px 0 4px 6px;
      border-color: transparent transparent transparent currentColor;
      transform: translate(-30%, -50%);
      transition: transform 0.15s;
    }
    &.expanded::before {
      transform: translate(-50%, -30%) rotate(90deg);
    }
  }
  @media (prefers-reduced-motion: reduce) {
    .toggle::before {
      transition: none;
    }
  }
</style>
