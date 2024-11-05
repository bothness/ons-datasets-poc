<script>
  import { createEventDispatcher } from "svelte";

  export let page = 1;
  export let pages = 10;

  const dispatch = createEventDispatcher();
  
  function select(p) {
    page = p;
    dispatch('select', page);
  }

  function getMid(p, max) {
    if (max < 2) return [];
    let mid;
    if (p < 4) mid = [2, 3, 4];
    else if (p > max - 3) mid = [max - 3, max - 2, max - 1];
    else mid = [p - 1, p, p + 1];
    if (mid[0] <= 1) return mid.slice(mid.indexOf(1));
    if (mid[2] >= pages) return mid.slice(0, mid.indexOf(max));
    return mid;
  }

  $: midPages = getMid(page, pages);
</script>

<nav class="ons-pagination ons-pagination--no-indicator" aria-label="Pagination (Page {page} of {pages})">
  <div class="ons-pagination__position">Page {page} of {pages}</div>
  <ul class="ons-pagination__items">
    {#if page !== 1}
    <li class="ons-pagination__item ons-pagination__item--previous">
      <a on:click|preventDefault={() => select(page - 1)} href="#prev" class="ons-pagination__link ons-pagination__link--no-underline" rel="prev"
        aria-label="Go to the previous page (Page 4)"><svg class="ons-icon ons-icon--m ons-u-mr-2xs" viewBox="0 0 17 13"
          xmlns="http://www.w3.org/2000/svg" focusable="false" fill="currentColor" role="img"
          title="ons-icon-arrow-previous">
          <path
            d="M6.4.2.3 6.4c-.2.2-.2.4 0 .6l6.2 5.8c.2.2.4.1.6 0l.8-.9c.2-.2.1-.4 0-.6l-4-3.7h12.5c.2 0 .4-.2.4-.4V6c0-.2-.2-.4-.4-.4H3.8l4-4c.2-.1.2-.4.1-.5L7 .2c-.1-.1-.4-.1-.6 0z" />
        </svg><span class="ons-pagination__link-text">Previous</span>
      </a>
    </li>
    {/if}
    <li class="ons-pagination__item" class:ons-pagination__item--current={page === 1}>
      <a on:click|preventDefault={() => select(1)} href="#1" class="ons-pagination__link" aria-label="Go to the first page (Page 1)">1</a>
    </li>
    {#if pages > 5 && page > 3}
      <li class="ons-pagination__item ons-pagination__item--gap">&hellip;</li>
    {/if}
    {#each midPages as p}
    <li class="ons-pagination__item" class:ons-pagination__item--current={page === p}>
      <a on:click|preventDefault={() => select(p)} href="#{p}" class="ons-pagination__link"  aria-label="Go to page {p}" rel="prev">{p}</a>
    </li>
    {/each}
    {#if pages > 5 && page < pages - 2}
      <li class="ons-pagination__item ons-pagination__item--gap">&hellip;</li>
    {/if}
    <li class="ons-pagination__item" class:ons-pagination__item--current={page === pages}>
      <a on:click|preventDefault={() => select(pages)} href="#{pages}" class="ons-pagination__link" aria-label="Go to the last page (Page {pages})">{pages}</a>
    </li>
    {#if page !== pages}
    <li class="ons-pagination__item ons-pagination__item--next">
      <a on:click|preventDefault={() => select(page + 1)} href="#next" class="ons-pagination__link ons-pagination__link--no-underline" rel="next"
        aria-label="Go to the next page (Page 6)">
        <span class="ons-pagination__link-text">Next</span><svg class="ons-icon ons-icon--m ons-u-ml-2xs"
          viewBox="0 0 17 13" xmlns="http://www.w3.org/2000/svg" focusable="false" fill="currentColor" role="img"
          title="ons-icon-arrow-next">
          <path
            d="m10 .2-.9.9c-.1.1-.1.4 0 .5l4 4H.6c-.2 0-.4.2-.4.4v1.2c0 .2.2.4.4.4h12.5l-3.9 3.7c-.2.2-.2.4 0 .6l.8.9c.2.2.4.2.6 0L16.8 7c.2-.2.2-.4 0-.6L10.7.3c-.3-.2-.5-.2-.7-.1z" />
        </svg></a>
    </li>
    {/if}
  </ul>
</nav>