<script>
	export let item;
	export let filterText;

	const capitalise = (str) => str[0].toUpperCase() + str.slice(1);

	const markFilter = (str, filterText) => filterText ? str.replace(new RegExp(filterText, "gi"), (txt) => `<mark>${txt}</mark>`) : str;
</script>

<li class="ons-document-list__item">
	<div class="ons-document-list__item-content">
		<div class="ons-document-list__item-header">
			<h2 class="ons-document-list__item-title ons-u-fs-m ons-u-mt-no ons-u-mb-2xs">
				<a href="https://www.ons.gov.uk/datasets/{item.id}" target="_blank">{@html markFilter(item.title, filterText)}</a>
			</h2>
			<ul class="ons-document-list__item-metadata ons-u-mb-2xs">
				{#if item.topic}
					<li class="ons-document-list__item-attribute">
						<span class="ons-u-fw-b">Topic: </span>
						<span>{capitalise(item.topic)}</span>
					</li>
				{/if}
				<li class="ons-document-list__item-attribute">
					<span class="ons-u-fw-b">Released: </span>
					<span>{item.date.toLocaleDateString("en-GB", {year: 'numeric', month: 'short', day: 'numeric'})}</span>
				</li>
				<li class="ons-document-list__item-attribute">
					<span class="ons-u-fw-b">Frequency: </span>
					<span>{item.frequency}</span>
				</li>
			</ul>
		</div>
		<div class="ons-document-list__item-description">
			<p>{item.description.length > 250 ? item.description.slice(0, 250) + ' [...]' : item.description}</p>
		</div>
	</div>
</li>