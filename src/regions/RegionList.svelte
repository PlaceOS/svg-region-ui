<script lang="ts">
    import Icon from '../ui/Icon.svelte';
    import { COLOURS } from '../common/globals';
    import type { MapRegion } from '../common/types';
    import type { Point } from '@yuion/svg-viewer';

    export let regions: MapRegion[] = [];
    export let top_left: Point = { x: .5, y: .5 };
    export let bottom_right: Point = { x: .5, y: .5 };

    let active_region = 0;

    $: {
        console.log('Reacted');
        if (!regions || regions.length < 1) {
            regions = [{ name: 'Area 1', capacity: 100, color: '#e66465', top: .5, left: .5, bottom: .5, right: .5 }]
        }
        const top = Math.floor(top_left.y * 1000) / 1000;
        const left = Math.floor(top_left.x * 1000) / 1000;
        const right = Math.floor(bottom_right.x * 1000) / 1000;
        const bottom = Math.floor(bottom_right.y * 1000) / 1000;
        const points = [
            [left, top],
            [right, top],
            [right, bottom],
            [left, bottom]
        ]
        regions[active_region].points = points;
        regions[active_region].height = Math.abs(top - bottom);
        regions[active_region].width = Math.abs(left - right);
        regions[active_region].location = {
            x: (left + right) / 2,
            y: (top + bottom) / 2
        };
        regions = regions;
    }

    function newRegion() {
        regions = regions.concat([{
            name: `Area ${regions.length + 1}`,
            color: COLOURS[regions.length % COLOURS.length],
            capacity: 100,
            top: .5,
            left: .5,
            bottom: .5,
            right: .5,
            height: 0,
            width: 0,
            content: document.createElement('div')
        } as any]);
        regions = regions;
        setActive(regions.length - 1);
        top_left = { x: .5, y: .5 };
        bottom_right = { x: .5, y: .5 };
    }

    function setActive(i: number) {
        if (i === active_region) return;
        const first_point = (regions[i].points || [])[0] || [.5, .5];
        const third_point = (regions[i].points || [])[2] || [.5, .5];
        top_left = { x: first_point[0], y: first_point[1] };
        bottom_right = { x: third_point[0], y: third_point[1] };
        active_region = i;
    }

</script>
<style>
    [name="color"] {
        height: 1em;
        min-width: 1em;
        width: 1em;
    }

    [name="region-name"] {
        min-width: 4em;
    }

    li {
        border-bottom: 1px solid #f0f0f0;
    }

    li:last-child {
        border: none;
    }

    [name="badge"] {
        background-color: #4caf50;
    }
</style>

<ul name="list" class="rounded list-none m-4 bg-white shadow text-black overflow-auto flex-1">
    <li on:click={newRegion}>
        <button class=" rounded p-2 flex items-center w-full hover:bg-gray-100">
            <Icon klass="material-icons" content="add"/>
            New Area
        </button>
    </li>
    {#each regions as region, i }
        <li class="p-2 flex items-center flex-wrap hover:bg-gray-100 cursor-pointer" on:click={e => setActive(i)}>
            <input type="number" name="capacity" class="rounded w-16 px-3 mr-2 py-2" bind:value={region.capacity} placeholder="Capacity" />
            <input type="color" name="color" class="rounded" bind:value={region.color} />
            <input type="text" name="region-name" class="px-3 mx-2 py-2 flex-1 text-base bg-transparent" bind:value={region.name} placeholder="Region Name" />
            <div class="text-xs flex items-center w-full text-gray-500 justify-end">
                {#if i === active_region}
                    <div name="badge" class="px-2 py-1 rounded text-white">Active</div>
                {/if}
                <div class="w-0 flex-1"></div>
                <div class="leading-tight">
                    <span>{'{'} x: {region.points[0][0]}, y: {region.points[0][1]} {'}'}</span>
                    <br/>
                    <span>{'{'} x: {region.points[3][0]}, y: {region.points[3][1]} {'}'}</span>
                </div>
            </div>
        </li>
    {/each}
</ul>
