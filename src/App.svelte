<script>
	import { onMount } from "svelte";
	import {
    PhaseBanner,
    Header,
		Section,
		Notice,
		Twisty,
		List,
    NavSections,
		Accordion,
		AccordionItem,
		Dropdown,
		Input,
		Checkboxes,
    Footer,
  } from "@onsvisual/svelte-components";
  import "@onsvisual/svelte-components/css/main.css";
	import DocumentList from "./lib/DocumentList.svelte";
	import Document from "./lib/Document.svelte";
  import Paginator from "./lib/Paginator.svelte";
	import DateInput from "./lib/DateInput.svelte";

	let datasets, filters;
	let topic = [];
	let frequency = [];
	let coverage = [];
	let granularity = [];
	let date;
  let filterText;
  let page = 1;
  let sort = false;
	let reset;

	const capitalise = (str) => str[0].toUpperCase() + str.slice(1);
	const countries = {E: "England", N: "Northern Ireland", S: "Scotland", W: "Wales"};

	function addFauxDates(items) {
		const freqs = ["Weekly", "Monthly", "Quarterly", "Annual", "Biennial", "Decennial"];
		const date = +(new Date());
		const day = 60 * 60 * 24 * 1000;
		const dates = [date, date - day * 7, date - day * 30, date - day * 90, date - day * 365, date - day * 730].map(d => new Date(d));
		const indices = freqs.map(freq => {
			const frqs = items.map(d => d.frequency);
			return frqs.slice(0, frqs.lastIndexOf(freq)).lastIndexOf(freq);
		});
		console.log(dates, indices)
		return items.map((d, i) => {
			d.date = i <= indices[0] ? dates[0] :
				i <= indices[1] ? dates[1] :
				i <= indices[2] ? dates[2] :
				i <= indices[3] ? dates[3] :
				i <= indices[4] ? dates[4] :
				dates[5];
			return d;  
		});
	}
	
	async function fetchDatasets() {
		const res = await fetch("https://api.beta.ons.gov.uk/v1/datasets?limit=500");
		const ds = await res.json();
		ds.items = ds.items.map(d => {
			d.topic = d.survey || d?.keywords?.[0];
			d.frequency = d.survey === "census" ? "Decennial" :
				!d.release_frequency || d.release_frequency.startsWith("To") ? null :
				d.release_frequency.startsWith("Annual") ? "Annual" :
				d.release_frequency.startsWith("Quarterly") ? "Quarterly" :
				d.release_frequency;
			if (d.frequency === null) d.frequency = d.title.startsWith("Quarterly") ? "Quarterly" : "Ad hoc";
			d.coverage = []
			d.granularity = ["National"];
			if (d.survey === "census") {
				d.coverage = ["E", "W"];
				d.granularity = ["National", "Regional", "Local authority", "Small area"];
			} else {
				if (d.description.includes("United Kingdom") || d.description.includes("UK")) {
					d.coverage = ["E", "N", "S", "W"];
				} else if (d.description.includes("Great Britain")) {
					d.coverage = ["E", "S", "W"];
				} else {
					if (d.description.includes("England")) d.coverage.push("E");
					if (d.description.includes("Northern Ireland")) d.coverage.push("N");
					if (d.description.includes("Scotland")) d.coverage.push("S");
					if (d.description.includes("Wales")) d.coverage.push("W");
					if (!d.coverage[0]) d.coverage = ["E", "W"];
				}
				if (d.description.includes("local authorit")) d.granularity = [...d.granularity, "Regional", "Local authority"];
				else if (d.description.includes("region")) d.granularity.push("Regional");
			}
			d.coverage = d.coverage.map(d => countries[d]);
			return d;
		});
		ds.items = addFauxDates(ds.items);
		console.log({ds})
		return ds;
	}

	function filter(datasets, filters, date, filterText, sort) {
    page = 1;
		if (!datasets) return null;
		const ftr = [];
		for (const key of Object.keys(filters)) {
			if (filters[key][0]) ftr.push({key, filter: filters[key]});
		}
    let items = filterText ? datasets.items.filter(d => d.title.toLowerCase().includes(filterText.toLowerCase())) : [...datasets.items];
		if (date) items = items.filter(d => d.date < date);
    if (sort) items.sort((a, b) => a.title.localeCompare(b.title));
		if (!ftr[0]) return items;
		return items.filter(d => ftr.every(f => {
			if (Array.isArray(d[f.key])) return f.filter.some(key => d[f.key].includes(key));
			return f.filter.includes(d[f.key]);
		}));
	}

  function clearFilters() {
    topic = [];
    frequency = [];
    coverage = [];
    granularity = [];
    filterText = "";
		date = null;
    page = 1;
		reset = !reset;
  }

	onMount(async () => {
		datasets = await fetchDatasets();
		const topic = datasets.items.filter(d => d.topic).map(d => d.topic);
		const frequency = datasets.items.filter(d => d.frequency).map(d => d.frequency);
		filters = {
			topic: Array.from(new Set(topic))
				.sort((a, b) => a.localeCompare(b))
				.map(d => ({id: d, label: `${capitalise(d)} (${topic.filter(t => t === d).length})`})),
			frequency: Array.from(
				new Set(frequency)
			).sort((a, b) => a.localeCompare(b)).map(d => ({id: d, label: `${capitalise(d)} (${frequency.filter(t => t === d).length})`})),
			coverage: Object.values(countries).map(d => ({id: d, label: d})),
			granularity: ["National", "Regional", "Local authority", "Small area"].map(d => ({id: d, label: d}))
		};
	});

	$: items = filter(datasets, {topic, frequency, coverage, granularity}, date, filterText, sort);
</script>

<svelte:head>
	<link rel="stylesheet" href="https://unpkg.com/@onsvisual/svelte-components@latest/dist/css/main.css"/>
</svelte:head>

<PhaseBanner phase="prototype"/>
<Header title="Data catalogue POC" compact />

<Section marginTop>
	<Notice>
		<p style:margin-bottom="18px">This is a proof-of-concept (POC) prototype for a dataset catalogue page based on the datasets within the <a href="https://developer.ons.gov.uk/dataset/" target="_blank">legacy ONS beta API</a>. The layout broadly follows existing catalogue pages and the ONS design system. It is also intended to highlight some of the limitations of the legacy API.</p>
	<Twisty title="API limitations">
		<p>A number of workarounds were required to make this prototype run from the API:</p>
		<List>
			<li>Since the API has no built-in filter or sort capabilities, the full list of datasets gets requested at once, and filtering and sorting happens client-side.</li>
			<li>Geographic coverage and granularity are inferred from the dataset description, since they are not included in the metadata model.</li>
			<li>Frequency ("quarterly", "annual" etc.) is cleaned up, since the field is not consistently used across all datasets.</li>
			<li>Latest release date is a very rough approximation, since the actual date can only be requested for an individual version of an individual dataset (which would therefore require hundreds of API requests).
		</List>
	</Twisty>
	</Notice>
</Section>

<NavSections contentsLabel="Filter results">
	<div slot="before-nav" style:position="absolute" style:right="20px">
		<a on:click|preventDefault={clearFilters} href="#clear">Clear all</a>
	</div>
	<div slot="after-nav" style:padding-right="20px">
		{#if filters}
		{#key reset}
			<Accordion>
				<AccordionItem title="Search" open>
					<Input label="Search keywords" bind:value={filterText}/>
				</AccordionItem>
				<AccordionItem title="Topic">
					<Checkboxes items={filters.topic} bind:value={topic} compact/>
				</AccordionItem>
				<AccordionItem title="Coverage">
					<Checkboxes items={filters.coverage} bind:value={coverage} compact/>
				</AccordionItem>
				<AccordionItem title="Granularity">
					<Checkboxes items={filters.granularity} bind:value={granularity} compact/>
				</AccordionItem>
				<AccordionItem title="Frequency">
					<Checkboxes items={filters.frequency} bind:value={frequency} compact/>
				</AccordionItem>
				<AccordionItem title="Release date">
					<DateInput bind:date/>
				</AccordionItem>
			</Accordion>
		{/key}
		{/if}
	</div>
	{#if items}
		<div class="search__count">
			<h2>
				<span class="search__summary__count">{items.length}</span>
				results
			</h2>
		</div>
    <div class="ons-u-mt-s ons-u-mb-s ons-u-pb-s ons-u-bb">
      <div class="ons-field--inline">
        <label class="ons-label" for="sort">Sort by</label>
        <select id="sort" name="sort" class="ons-input ons-input--select ons-u-wa--@xxs ons-u-mr-s ons-input--sort-select" bind:value={sort}>
          <option class="sort__option" value={false} selected="">
            Release date
          </option>
          <option class="sort__option" value={true}>
            Title
          </option>
        </select>
      </div>
    </div>
	  <DocumentList>
			{#each items.slice((page - 1) * 10, page * 10) as item}
				<Document {item} {filterText}/>
			{/each}
		</DocumentList>
    {#if items.length > 10}
      <Paginator bind:page pages={Math.ceil(items.length / 10)}/>
    {/if}
	{/if}
</NavSections>

<Footer compact />

<style>
	:global(nav.ons-toc > ol.ons-list) {
		display: none;
	}
	:global(.ons-details__content) {
		padding-bottom: 20px !important;
	}
	:global(#toc) {
		position: relative;
	}

	:global(aside.ons-toc-container) {
		margin-top: -14px !important;
	}
  :global(#search-content input[type=text]) {
    width: 100% !important;
    max-width: 100% !important;
  }
	:global(label.ons-label) {
		font-weight: normal;
	}
</style>