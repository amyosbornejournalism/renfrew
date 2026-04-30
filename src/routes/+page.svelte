<script lang="ts">
  import rawBlocks from '$lib/doc.blocks.json';
  import DocRenderer from '$lib/components/DocRenderer.svelte';
  import type { Block } from '$lib/components/DocRenderer.svelte';
  const blocks = rawBlocks as Block[];
  import { asset } from '$app/paths';
</script>


<div class="mainContainer">
  <DocRenderer {blocks} />
  <p class="smaller mb-1 mt-4 fw-bold">Credits</p>
  <p class="smaller">Reporting by Amy Osborne and Mariana Best. Photos, videos and design by Amy Osborne. Development by Amy Osborne and Yoli Martinez.</p>
</div>


<!-- Hard-code any custom code that should appear AFTER Google Doc below here. -->
<footer class="container-fluid bg-dark text-white p-5">
  <p class="text-center">UC Berkeley Journalism</p>
</footer>