<script lang="ts">
  /**
   * Generic embedable media, inserted as HTML
   * 
   * Example shortcode usage:
   * [[EmbedCode size="large"]]
   *   <iframe src="https://example.com/"></iframe>
   *   <p>Other random HTML code</p>
   * [[/EmbedCode]]
   */

  import { browser } from "$app/environment";
  import { onMount, tick } from "svelte";
  import { asset } from '$app/paths';
  import image from '../assets/101125_Hafsa_253.jpg';

  export let bodyHtml: string | undefined;
  export let size: "full" | "large" | "fit" = "large";

  const allowedSizes = new Set(["full", "large", "fit"]);
  $: normalizedSize = allowedSizes.has(size) ? size : "large";

  // Decodes common entities, numeric entities (decimal + hex),
  // and handles double-escaped strings by iterating a few times.
  // Also strips a few common invisible/junk characters WYSIWYGs add.
  function decodeWysiwygHtml(input: string): string {
    if (!input) return input;

    let s = String(input);

    // Normalize newlines early
    s = s.replace(/\r\n?/g, "\n");

    // Strip some invisible/junk chars that commonly sneak in
    // - zero-width space/joiners, BOM
    s = s.replace(/[\u200B-\u200D\uFEFF]/g, "");

    // Convert NBSP variants to normal spaces (keep if you prefer &nbsp;)
    s = s.replace(/\u00A0/g, " ");

    // Some editors paste “smart quotes” directly; not entities.
    s = s
      .replace(/[\u201C\u201D]/g, '"')
      .replace(/[\u2018\u2019]/g, "'");

    // Helper: decode ONE pass of entities
    const decodeOnce = (str: string) => {
      // 1) numeric decimal: &#123;
      str = str.replace(/&#(\d+);?/g, (_, dec) => {
        const code = Number(dec);
        if (!Number.isFinite(code) || code < 0 || code > 0x10ffff) return _;
        try {
          return String.fromCodePoint(code);
        } catch {
          return _;
        }
      });

      // 2) numeric hex: &#x1F600;
      str = str.replace(/&#x([0-9a-fA-F]+);?/g, (_, hex) => {
        const code = parseInt(hex, 16);
        if (!Number.isFinite(code) || code < 0 || code > 0x10ffff) return _;
        try {
          return String.fromCodePoint(code);
        } catch {
          return _;
        }
      });

      // 3) named entities. Do &amp; LAST-ish? Actually we want to decode &amp; too, but if we do it
      // before others, we can turn &amp;lt; into &lt; which will decode next iteration.
      // That’s why we iterate decodeOnce a few times below.
      const named: Record<string, string> = {
        lt: "<",
        gt: ">",
        quot: '"',
        apos: "'",
        amp: "&",
        nbsp: " ",
        ndash: "–",
        mdash: "—",
        hellip: "…",
        bull: "•",
        middot: "·",
        copy: "©",
        reg: "®",
        trade: "™",
        euro: "€",
        pound: "£",
        yen: "¥",
        cent: "¢",
      };

      str = str.replace(/&([a-zA-Z]+);/g, (m, name) => {
        const key = name.toLowerCase();
        return key in named ? named[key] : m;
      });

      // Common edge case: &amp without semicolon
      str = str.replace(/&amp(?![a-zA-Z0-9#])/g, "&");

      return str;
    };

    // Iterate a few times to handle double/triple escaping:
    // e.g. &amp;lt;iframe&amp;gt; -> &lt;iframe&gt; -> <iframe>
    for (let i = 0; i < 4; i++) {
      const next = decodeOnce(s);
      if (next === s) break;
      s = next;
    }

    // Optional: unwrap common “pasted as code” wrappers
    // <pre><code> ... </code></pre>
    const preCodeMatch = s.match(/^\s*<pre[^>]*>\s*<code[^>]*>([\s\S]*?)<\/code>\s*<\/pre>\s*$/i);
    if (preCodeMatch) s = preCodeMatch[1];

    // Optional: unwrap markdown fences if someone pasted a fenced block into a rich text field
    // ```html ... ```
    const fenceMatch = s.match(/^\s*```[a-zA-Z]*\s*\n([\s\S]*?)\n```\s*$/);
    if (fenceMatch) s = fenceMatch[1];

    return s.trim();
  } 


  function normalizeInBrowser(html: string): string {
    const doc = new DOMParser().parseFromString(html, "text/html");
    return doc.body.innerHTML;
  }

  $: normalizedSrc =
    bodyHtml
      ? (browser ? normalizeInBrowser(decodeWysiwygHtml(bodyHtml)) : decodeWysiwygHtml(bodyHtml))
      : undefined;


  let scroller: HTMLElement | null = null;
  let getIframe: HTMLIFrameElement | null = null;
  let blur = 30;
  let opacity = 0.3;
  let canPlay = false;
  let ticking = false;
  

  function handleScroll() {
    if (!scroller || !getIframe) return;

    const rect = scroller.getBoundingClientRect();
    const vh = window.innerHeight;
    let progress = (vh - rect.top) / (rect.height + vh);


    if (progress < 0.55) {
      blur = 30;
      opacity = 0.3
      canPlay = false;
      getIframe.style.opacity = "0";
    } else {
      const start = 0.55;
      const end = 0.7;
      const t = (progress - start) / (end - start);

      canPlay = false;
      blur = Math.max(30 - t * 30, 0);
      opacity = 0.3 + t * 0.6;
      getIframe.style.opacity = "0";

      if (blur === 0) {
        canPlay = true;
        opacity = 1;
        getIframe.style.opacity = "1";
      }
    }
  }

  function updateScroll() {
    if (!ticking) {
      requestAnimationFrame(() => {
        handleScroll();
        ticking = false;
      })
      ticking = true;
    }
  }

  onMount(() => {
    getIframe = document.querySelector(".headScroller .bg .videoWrap iframe");
    window.addEventListener("scroll", updateScroll, { passive: true });
    handleScroll();

    return () => window.removeEventListener("scroll", handleScroll);
  });
</script>


{#if normalizedSrc}

<div class="headScroller" bind:this={scroller}>
  <div 
    class="bg"
    class:active={canPlay}
    // style="--blur: {blur}px"
  >
    <div class="videoWrap">
      <figure class="full-bleed video-section">
          <img src={image} alt="test" style="--blur: {blur}px; --opacitySlide: {opacity}; {canPlay ? "z-index: 0;" : "z-index: 10;"}"/>
          {@html normalizedSrc} 
      </figure>
    </div>
  </div>
  <div class="sliderContent">
    <div class="slide fade-in" style="font-weight: 600;">
      <p class="pb-0">They went to The Renfrew Center for help.<br>Many left with trauma.</p>
    </div>
    <div class="slide">
      <p class="fw-bold">How does a place built to heal end up harming?</p>
    </div>
    <div class="slide">
      <h1 style="font-weight: 600;">Trauma In Treatment </h1>
      <p class="fs-4 fw-light">A leading eating disorder center markets "gold standard" recovery, but patient outcomes tell a different story.</p>
      <p class="fs-6 fw-light">By Amy Osborne and Mariana Best<br>May 5, 2026</p>

      <!-- <p>"She was going in there willingly. And you make her feel so bad that she feels like she has no other choice than to kill herself."</p> -->
    </div>
    <div class="slide"></div>
  </div>
</div>

{/if}