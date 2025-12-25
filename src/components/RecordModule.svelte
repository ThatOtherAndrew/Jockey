<script lang="ts">
    import { VinylRecord, PlaybackController } from 'virtual-vinyl';

    let speed = $state(0);
    const controller = new PlaybackController();

    $effect(() => {
        controller.loadFromUrl(
            'https://cdn.freesound.org/previews/63/63054_216886-hq.mp3',
            true,
        );
    });
</script>

<div class="bg-red-500">
    <VinylRecord {controller} bind:speed img="https://picsum.photos/200" />
    Speed: <input type="number" bind:value={speed} />
</div>
