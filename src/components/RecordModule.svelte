<script lang="ts">
    import { VinylRecord, PlaybackController } from 'virtual-vinyl';

    const API_KEY = 'CrsqruTkdqIi3nB8SSVIdgpIgpM3ilw6xC9B5iJw'; // yes this is an atrocious idea but whatever

    type Sample = {
        title: string;
        url: string;
        duration: number;
        id: number;
    };

    let speed = $state(0);
    let searchSample = $state('');
    let searchPromise: Promise<Sample[]> | undefined = $state();
    let sampleUrl: string | undefined = $state();

    const controller = new PlaybackController();

    $effect(() => {
        if (sampleUrl) {
            controller.loadFromUrl(sampleUrl, true);
        }
    });

    async function searchFreesound(
        query: string,
        limit = 5,
    ): Promise<Sample[]> {
        const fields = 'id,name,previews,duration';
        const baseUrl = 'https://freesound.org/apiv2/search/text/';

        const params = new URLSearchParams({
            query: query,
            token: API_KEY,
            fields: fields,
            page_size: limit.toString(),
        });

        try {
            const response = await fetch(`${baseUrl}?${params}`);
            if (!response.ok)
                throw new Error(`Freesound API Error: ${response.status}`);

            const data = await response.json();

            return data.results.map((sound: any) => ({
                title: sound.name,
                // Prefer High-Quality MP3, fall back to LQ if missing
                url:
                    sound.previews['preview-hq-mp3'] ||
                    sound.previews['preview-lq-mp3'],
                duration: sound.duration,
                id: sound.id,
            }));
        } catch (e) {
            console.error('Freesound Search Failed:', e);
            return [];
        }
    }
</script>

<div>
    <VinylRecord {controller} bind:speed img="https://picsum.photos/200" />

    <input type="search" list="samples" bind:value={searchSample} />
    <button
        onclick={() => {
            searchPromise = searchFreesound(searchSample);
        }}>Search</button
    >
    <select bind:value={sampleUrl}>
        {#if searchPromise}
            {#await searchPromise}
                <option>Searching...</option>
            {:then samples}
                {#each samples as sample}
                    <option value={sample.url}>{sample.title}</option>
                {/each}
            {:catch}
                <option>Search error</option>
            {/await}
        {:else}
            <option>Select a sample...</option>
        {/if}
    </select>

    <br />

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
