<script>
    import { getContext, onMount, onDestroy, afterUpdate } from "svelte";
    import { cubicOut } from "svelte/easing";
    import { tweened } from "svelte/motion";

    export let x;
    export let y;
    export let r;
    export let fill;
    export let contextName = "canvas";

    const { register, deregistor, invalidate } = getContext(contextName);

    const tweeneParams = {
        duration: 1000,
        easing: cubicOut,
    };
    const tX = tweened(undefined, tweeneParams);
    const tY = tweened(undefined, tweeneParams);
    const tR = tweened(undefined, tweeneParams);

    function draw(ctx) {
        ctx.translate($tX, $tY);
        ctx.globalAlpha = 0.8;
        ctx.fillStyle = fill;
        ctx.beginPath();
        ctx.arc(0, 0, $tR, 0, 2 * Math.PI, false);
        ctx.fill();
    }
    onMount(() => {
        register(draw);
        invalidate();
    });

    onDestroy(() => {
        deregistor(draw);
    });

    // afterUpdate(() => {
    //     invalidate();
    // });
    afterUpdate(invalidate);

    $: tX.set(x);
    $: tY.set(y);
    $: tR.set(r);
</script>
