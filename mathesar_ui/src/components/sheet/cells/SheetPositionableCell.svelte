<script lang="ts">
  import { type ColumnPosition, getSheetContext } from '../utils';

  const { stores } = getSheetContext();
  const { columnStyleMap } = stores;

  export let index: number;
  export let columnSpan = 1;
  /** When true, keeps the cell pinned to the left while horizontally scrolling */
  export let stickToLeft = false;

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
      // Inline styles here intentionally override default absolute positioning
      // to pin the element to the sheet's left viewport edge while preserving width.
      return `position: sticky; left: 0; width: ${width}px; z-index: var(--z-index__sheet__positionable-cell);`;
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
