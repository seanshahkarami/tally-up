<script lang="ts">
    import "../app.css";
    import QrCode from "../components/QRCode.svelte";

    const isBrowser = !(typeof window === "undefined");

    type State = {
        name: string;
        total: number;
        current: number;
        version: number;
    };

    function getStateFromStorage(): State | null {
        const storageState = localStorage.getItem("state");
        if (storageState == null) {
            return null;
        }
        const [total, current, version] = storageState.split(",").map(Number);
        return { name: "storage", total, current, version };
    }

    function getStateFromUrl(): State | null {
        const urlParams = new URLSearchParams(window.location.search);
        const paramState = urlParams.get("state");
        if (paramState == null) {
            return null;
        }

        const [total, current, version] = paramState.split(",").map(Number);
        return { name: "url", total, current, version };
    }

    function getLatestState(): State {
        let states: State[] = [
            { name: "default", total: 0, current: 0, version: 0 },
        ];

        const storageState = getStateFromStorage();
        if (storageState) {
            states.push(storageState);
        }

        const urlState = getStateFromUrl();
        if (urlState) {
            states.push(urlState);
        }

        // Return the newest state.
        return states.sort((a, b) => b.version - a.version)[0];
    }

    let total = $state(0);
    let current = $state(0);
    let version = $state(0);
    let isSharing = $state(false);

    let address = "";

    if (isBrowser) {
        address = `${window.location.protocol}//${window.location.hostname}:${window.location.port}`;
    }

    let shareText = $derived(
        `${address}/?state=${total},${current},${version}`,
    );
    let audioCtx: AudioContext | null = null;

    if (isBrowser) {
        const latestState = getLatestState();
        console.log("Initial state is", latestState.name);
        total = latestState.total;
        current = latestState.current;
        version = latestState.version;

        audioCtx = new window.AudioContext();
    }

    function playBeep(duration = 50, freq = 440) {
        if (audioCtx == null) {
            return;
        }
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();

        osc.type = "square";
        osc.frequency.value = freq;
        gain.gain.value = 0.05; // volume

        osc.connect(gain).connect(audioCtx.destination);
        osc.start();

        setTimeout(() => {
            osc.stop();
        }, duration);
    }

    function inc() {
        total++;
        current++;
        version++;
        if (isBrowser) {
            localStorage.setItem("state", `${total},${current},${version}`);
        }
        playBeep(30, 440);
    }

    function dec() {
        if (current == 0) {
            return;
        }
        current--;
        version++;
        if (isBrowser) {
            localStorage.setItem("state", `${total},${current},${version}`);
        }
        playBeep(30, 220);
    }

    let resetInterval: NodeJS.Timeout | null = $state(null);
    let resetCountdown = $state(0);

    function startResetTimer() {
        resetCountdown = 10;

        resetInterval = setInterval(() => {
            if (resetCountdown > 0) {
                resetCountdown--;
                return;
            }
            clearResetTimer();
            total = 0;
            current = 0;
            version = 0;
            if (isBrowser) {
                localStorage.setItem("state", `${total},${current},${version}`);
            }
        }, 1000);
    }

    function clearResetTimer() {
        if (resetInterval == null) {
            return;
        }
        clearTimeout(resetInterval);
        resetInterval = null;
    }
</script>

{#if isSharing}
    <div class="fixed inset-0 flex items-center justify-center">
        <div class="text-4xl leading-normal">
            <QrCode text={shareText} />
        </div>
    </div>
{:else}
    <div class="fixed inset-0 flex items-center justify-center">
        <div class="text-4xl leading-normal">
            {#if resetInterval != null}
                <p class="text-lg">
                    Hold to reset tally in {resetCountdown} seconds...
                </p>
            {:else}
                <p><strong>{total}</strong> <small>total</small></p>
                <p><strong>{current}</strong> <small>current</small></p>
            {/if}
        </div>
    </div>

    <button
        class="bg-red-400 w-24 h-24 rounded-full text-4xl fixed top-6 left-6 select-none"
        onclick={dec}>-</button
    >

    <button
        class="bg-green-400 w-24 h-24 rounded-full text-4xl fixed bottom-6 right-6 select-none"
        onclick={inc}>+</button
    >

    <button
        class="bg-gray-400 w-24 h-24 rounded-full text-4xl fixed top-6 right-6 select-none"
        onmousedown={startResetTimer}
        ontouchstart={startResetTimer}
        onmouseup={clearResetTimer}
        onmouseleave={clearResetTimer}
        ontouchend={clearResetTimer}
        ontouchcancel={clearResetTimer}>c</button
    >
{/if}

<button
    class="bg-blue-400 w-24 h-24 rounded-full text-4xl fixed bottom-6 left-6 select-none"
    onclick={() => (isSharing = !isSharing)}>s</button
>
