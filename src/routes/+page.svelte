<script lang="ts">
    import "../app.css";

    const isBrowser = !(typeof window === "undefined");

    let total = $state(0);
    let current = $state(0);

    if (isBrowser) {
        total = parseInt(localStorage.getItem("total") || "0");
        current = parseInt(localStorage.getItem("current") || "0");
    }

    let audioCtx: AudioContext | null = null;

    if (isBrowser) {
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
        if (isBrowser) {
            localStorage.setItem("total", total.toString());
            localStorage.setItem("current", current.toString());
        }
        if (navigator.vibrate) {
            navigator.vibrate(30);
        }
        playBeep(30, 440);
    }

    function dec() {
        if (current == 0) {
            return;
        }
        current--;
        if (isBrowser) {
            localStorage.setItem("current", current.toString());
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
