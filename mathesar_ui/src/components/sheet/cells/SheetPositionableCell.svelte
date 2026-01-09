<script lang="ts">
  import { type ColumnPosition, getSheetContext } from '../utils';
  import { onMount, onDestroy } from 'svelte';

  const { stores } = getSheetContext();
  const { columnStyleMap } = stores;

  export let index: number;
  export let columnSpan = 1;
  /** When true, keeps the cell pinned to the left while horizontally scrolling */
  export let stickToLeft = false;

  let viewportWidth = 0;
  let resizeObserver: ResizeObserver | null = null;

  function updateViewportWidth() {
    const el = document.querySelector<HTMLElement>("[data-sheet-body-element='list']");
    viewportWidth = el?.clientWidth ?? 0;
  }

  onMount(() => {
    updateViewportWidth();
    const el = document.querySelector<HTMLElement>("[data-sheet-body-element='list']");
    if (el && 'ResizeObserver' in window) {
      resizeObserver = new ResizeObserver(() => updateViewportWidth());
      resizeObserver.observe(el);
    } else {
      window.addEventListener('resize', updateViewportWidth);
    }
  });

  onDestroy(() => {
    if (resizeObserver) {
      resizeObserver.disconnect();
      resizeObserver = null;
    } else {
      window.removeEventListener('resize', updateViewportWidth);
    }
  });

  function calculateStyle(
    _columnStyleMap: Map<unknown, ColumnPosition>,
    _index: number,
    _columnSpan: number,
  ): string {
    const columnStyleValues = [..._columnStyleMap.values()];
    const startIndex = columnStyleValues[index] ? index : 0;
    const left = columnStyleValues[startIndex]?.left ?? 0;

    let endEntry = columnStyleValues[startIndex + _columnSpan - 1];
    endEntry = endEntry ?? columnStyleValues[columnStyleValues.length - 1];
    const width = endEntry ? endEntry.left + endEntry.width - left : 0;

    if (stickToLeft) {
      const w = viewportWidth ? Math.min(width, viewportWidth) : width;
      return `position: sticky; left: 0; width: ${w}px; z-index: var(--z-index__sheet__positionable-cell);`;
    }
    return `left: ${left}px; width: ${width}px`;
  }

  $: style = calculateStyle($columnStyleMap, index, columnSpan);
</script>

<div data-sheet-element="positionable-cell" {style}>
  <slot />
</div>

<style>
  [data-sheet-element='positionable-cell'] {
    position: absolute;
    top: 0;
    height: 100%;
  }
</style>
