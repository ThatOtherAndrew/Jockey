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

<div>
    <VinylRecord {controller} bind:speed img="https://picsum.photos/200" />
    <input
        type="range"
        min="-5"
        max="5"
        step="0.00000000001"
        defaultvalue="0"
        bind:value={speed}
        oninput={() => controller.resumePlayback()}
    />
    <input
        type="number"
        min="-5"
        max="5"
        step="0.01"
        defaultvalue="0"
        bind:value={speed}
        oninput={() => controller.resumePlayback()}
    />
</div>
