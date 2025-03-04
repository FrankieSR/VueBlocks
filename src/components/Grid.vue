<template>
    <div
        ref="gridContainer"
        class="grid-container"
        :class="{ 'grid-active': isManipulating }"
        :style="{ '--grid-cell-size': `${gridCellSize}px` }"
        @dragover.prevent
        @mousemove="handlePointerMove"
        @touchmove.prevent="handlePointerMove"
        @mouseup="handlePointerUp"
        @touchend="handlePointerUp"
    >
        <slot :gridWidth="gridWidth" :gridHeight="gridHeight" :gridCellSize="gridCellSize" />
    </div>
</template>

<script lang="ts" setup>
    import { ref, provide, onMounted, onUnmounted } from 'vue';
    import { type GridContext, type Rect } from '@/types/gridTypes';

    const props = defineProps<{
        gridCellSize?: number;
    }>();

    const gridCellSize = ref(props.gridCellSize ?? 50);

    const gridContainer = ref<HTMLElement | null>(null);
    const gridWidth = ref<number>(0);
    const gridHeight = ref<number>(0);

    const isManipulating = ref<boolean>(false);
    const activeItemId = ref<string | null>(null);
    const activeItemRect = ref<Rect | null>(null);

    const activeItem = ref<{
        onMouseMove: (event: MouseEvent | TouchEvent) => void;
        onMouseUp: () => void;
    } | null>(null);

    let rAF: number | null = null;
    let resizeObserver: ResizeObserver | null = null;

    const updateSize = () => {
        if (gridContainer.value) {
            gridWidth.value = gridContainer.value.clientWidth;
            gridHeight.value = gridContainer.value.clientHeight;
        }
    };

    provide<GridContext>('gridContext', {
        gridContainer,
        gridWidth,
        gridHeight,
        gridCellSize,
        isManipulating,
        setActiveItem: (item: any) => {
            activeItem.value = item;
            activeItemId.value = item.id;
            activeItemRect.value = item.rect;
        },
        updateActiveItemRect: (rect: { x: number; y: number; w: number; h: number }) => {
            activeItemRect.value = rect;
        },
        clearActiveItem: () => {
            activeItem.value = null;
            activeItemId.value = null;
            activeItemRect.value = null;
        },
        activeItemId,
        activeItemRect,
    });

    const handlePointerMove = (event: MouseEvent | TouchEvent) => {
        if (rAF === null && activeItem.value) {
            rAF = requestAnimationFrame(() => {
                if (activeItem.value) {
                    activeItem.value.onMouseMove(event);
                }
                rAF = null;
            });
        }
    };

    const handlePointerUp = () => {
        if (activeItem.value) {
            activeItem.value.onMouseUp();
        }
    };

    onMounted(() => {
        if (gridContainer.value) {
            updateSize();
            resizeObserver = new ResizeObserver(() => {
                updateSize();
            });
            resizeObserver.observe(gridContainer.value);
        }
    });

    onUnmounted(() => {
        if (resizeObserver && gridContainer.value) {
            resizeObserver.unobserve(gridContainer.value);
        }
        activeItem.value = null;
    });
</script>

<style scoped>
    .grid-container {
        position: relative;
        width: 100%;
        height: 100%;
        transition: var(--grid-transition);
        background-color: var(--grid-bg-color);
    }

    .grid-active {
        background: var(--grid-active-bg-color);
        background-image:
            linear-gradient(
                0deg,
                transparent calc(var(--grid-cell-size) - 2px),
                var(--grid-line-color) calc(var(--grid-cell-size) - 1px),
                var(--grid-line-color) var(--grid-cell-size),
                transparent calc(var(--grid-cell-size) + 1px)
            ),
            linear-gradient(
                90deg,
                transparent calc(var(--grid-cell-size) - 2px),
                var(--grid-line-color) calc(var(--grid-cell-size) - 1px),
                var(--grid-line-color) var(--grid-cell-size),
                transparent calc(var(--grid-cell-size) + 1px)
            );
        background-size: var(--grid-cell-size) var(--grid-cell-size);
        box-shadow: var(--grid-active-inner-shadow);
    }
</style>
