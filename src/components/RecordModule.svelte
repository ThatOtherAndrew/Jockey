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
    let sampleUrl = $state('');

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

    <div class="my-3">
        <div class="mb-2 flex gap-2">
            <input
                class="flex-1 rounded border-3 px-4 py-2"
                type="search"
                list="samples"
                placeholder="Search for a sample..."
                bind:value={searchSample}
            />
            <button
                class="w-25 shrink-0 rounded bg-blue-500 px-4 py-2 font-bold text-white hover:bg-blue-700"
                onclick={() => {
                    searchPromise = searchFreesound(searchSample);
                }}>Search</button
            >
        </div>
        <select
            class="w-full rounded border-3 px-4 py-2"
            bind:value={sampleUrl}
        >
            {#if searchPromise}
                {#await searchPromise}
                    <option value="" disabled selected>Searching...</option>
                {:then samples}
                    <option value="" disabled selected
                        >{samples.length} samples found!</option
                    >
                    {#each samples as sample}
                        <option value={sample.url}>{sample.title}</option>
                    {/each}
                {:catch}
                    <option>Search error</option>
                {/await}
            {:else}
                <option value="" disabled selected>Select a sample...</option>
            {/if}
        </select>
    </div>

    <div class="flex gap-2">
        <input
            class="flex-1"
            type="range"
            min="-5"
            max="5"
            step="0.00000000001"
            defaultvalue="0"
            bind:value={speed}
            oninput={() => controller.resumePlayback()}
        />
        <input
            class="w-25 shrink-0 rounded bg-blue-500 px-4 py-2 font-bold text-white hover:bg-blue-700"
            type="number"
            min="-5"
            max="5"
            step="0.01"
            defaultvalue="0"
            bind:value={speed}
            oninput={() => controller.resumePlayback()}
        />
    </div>
</div>
