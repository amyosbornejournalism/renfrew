<script lang="ts">
  /**
   * Example shortcode usage:
   * [[CoolNewFeature src="https://example.com/embed" size="large"]]
   */
  
  export let src: string | undefined;
  export let size: 'full' | 'large' | 'fit' = 'large';
</script>

{#if src}
  {#if size === 'full'}
  

    <div style="border: 1px solid #000000; position: relative; padding-bottom: 53.25%; width:100%; padding-top: 30px; height: 0; overflow: hidden;">
      <iframe 
        style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; border: none;" 
        src="{src}"
        title="Video" 
        width="100%" height="100%" allowfullscreen="allowfullscreen"></iframe>
    </div>


  {:else if size === 'large'}

      <div style="margin:4rem 0; max-width: 80vw; border: 1px solid #000000; position: relative; padding-bottom: 40%; width:100%; padding-top: 30px; height: 0; overflow: hidden;">
        <iframe 
          style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; border: none;" 
          src="{src}" 
          title="Video"
          width="100%" height="100%" allowfullscreen="allowfullscreen"></iframe>
      </div>


  {:else}
    <!-- fit -->
    <div class="video-fit">
      <div style="border: 1px solid #000000; position: relative; padding-bottom: 53.25%; width:100%; padding-top: 30px; height: 0; overflow: hidden;">
        <iframe 
          style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; border: none;" 
          src="{src}" 
          title="Video"
          width="100%" height="100%" allowfullscreen="allowfullscreen"></iframe>
      </div>
    </div>

  {/if}
{/if}
