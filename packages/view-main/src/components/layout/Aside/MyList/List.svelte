<script lang="ts">
  import ListBranch from './ListBranch.svelte'
  import Menu from './Menu.svelte'
  import { useListItemHeight } from '@/modules/app/reactive.svelte'
  import type { Position } from '@/components/base/Menu.svelte'
  import { getAllList } from '@/modules/musicLibrary/store/actions'
  import { scrollPointerEvents } from '@/shared/compositions/scrollPointerEvents.svelte'
  import { verticalScrollbar } from '@/shared/compositions/verticalScrollbar.svelte'
  import { defaultLists, useUserList } from '@/modules/musicLibrary/reactive.svelte'
  import type { ComponentExports } from 'svelte'
  import { sortable } from '@/shared/compositions/sortable.svelte'
  import { updateUserListPosition } from './action'

  const listItemHeight = useListItemHeight(3.2)
  const picStyle = $derived(`height:${listItemHeight.val * 0.64}px; width:${listItemHeight.val * 0.64}px;`)

  let menu: ComponentExports<typeof Menu>

  // 只取顶层（parentId = null）；子列表由 ListBranch 按 parentId 递归取
  const userLists = useUserList(null)
  const lists = $derived([...$defaultLists, ...userLists.val])

  // 原来用 activeIndex，子列表没有全局下标，改用 id 标记当前右键项
  let activeId = $state<string | null>(null)

  const showMenu = (item: AnyListen.List.MyListInfo | null, position: Position) => {
    menu.show(item, position)
  }
</script>

<div class="list">
  {#await getAllList()}
    <div class="list-container tip">Loading...</div>
  {:then _}
    <div
      class="list-container my-list-container"
      role="list"
      tabindex="-1"
      {@attach scrollPointerEvents}
      {@attach verticalScrollbar({ offset: '0', scrollbarWidth: '0.375rem' })}
      oncontextmenu={(event) => {
        event.preventDefault()
        event.stopPropagation()
        activeId = null
        showMenu(null, { x: event.pageX, y: event.pageY })
      }}
    >
      <ul
        class="list"
        {@attach sortable({
          onupdate: (parentId, id, toTargetId, position) => {
            void updateUserListPosition(id, position - $defaultLists.length)
          },
          filter: 'default-list',
          activeElement: 'my-list-container',
        })}
      >
        {#each lists as item, index (item.id)}
          <li class="list-item draggable-item" class:default-list={item.type == 'default'} data-id={item.id}>
            <ListBranch
              listInfo={item}
              {index}
              {activeId}
              {picStyle}
              onmenu={(target, event) => {
                activeId = target.id
                showMenu(target, { x: event.pageX, y: event.pageY })
              }}
            />
          </li>
        {/each}
      </ul>
    </div>
  {:catch error}
    <div class="list-container tip">Load failed: {error.message}</div>
  {/await}
</div>

<Menu
  bind:this={menu}
  onhide={() => {
    activeId = null
  }}
/>

<style lang="less">
  .list {
    position: relative;
    height: 100%;
  }
  .list-container {
    position: relative;
    display: block;
    height: 100%;
    contain: strict;
  }
  .list-item {
    padding: 0 6px;
    -webkit-user-drag: revert-layer;
  }
  .tip {
    align-items: center;
    justify-content: center;
  }
</style>
