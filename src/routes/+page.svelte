<script lang="ts">
  import rawBlocks from '$lib/doc.blocks.json';
  import DocRenderer from '$lib/components/DocRenderer.svelte';
  import type { Block } from '$lib/components/DocRenderer.svelte';
  const blocks = rawBlocks as Block[];

  import { onMount } from "svelte";

  let scroller: HTMLElement | null = null;
  let bgVideo: HTMLVideoElement | null = null;

  let blur = 30;
  let canPlay = false;


  function handleScroll() {
    if (!scroller || !bgVideo) return;

    const rect = scroller.getBoundingClientRect();
    const vh = window.innerHeight;

    let progress = (vh - rect.top) / (rect.height + vh);

    if (progress < 0.55) {
      blur = 30;
      canPlay = false;
      bgVideo.pause();
    } else {
      canPlay = false;
      bgVideo.pause();
      const start = 0.55;
      const end = 0.70; // 👈 finish earlier

      const t = (progress - start) / (end - start);
      blur = Math.max(30 - t * 30, 0);

      if (blur === 0) {
        canPlay = true;
      }
    }
    
  }

  onMount(() => {
    window.addEventListener("scroll", handleScroll, { passive: true });
    handleScroll();

    return () => window.removeEventListener("scroll", handleScroll);
  });
</script>





<div class="mainContainer">
  <DocRenderer {blocks} />
</div>


<!-- Hard-code any custom code that should appear AFTER Google Doc below here. -->
<footer class="container-fluid bg-dark text-white p-5">
  <p class="text-center">See template on <a class="text-white" href="https://github.com/jrue/multimedia-template-2026" target="_blank">Github</a></p>
</footer>