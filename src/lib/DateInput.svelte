<script>
  export let date;

  let day = null;
  let month = null;
  let year = null;

  function updateDate(day, month, year) {
    let dt;
    if (year && year.length === 4 && typeof +year === 'number') dt = new Date(`${year}-${month || '01'}-${day || '01'}`);
    date = dt || null;
  }

  $: updateDate(day, month, year);
</script>

<fieldset id="before-date" class="ons-fieldset" data-invalid-range="Enter a released before year that is later than" data-invalid-date="Enter a real date">
  <legend class="ons-fieldset__legend ons-fieldset__legend--with-description">Released before</legend>
  <div class="ons-fieldset__description">For example: 2020 or 01 07 2020</div>
  <div class="ons-field-group">
    <div class="ons-field">
      <label class="ons-label" for="before-date-day">Day</label>
      <input type="text" id="before-date-day" class="ons-input ons-input--text ons-input-type__input ons-input-number--w-2" name="before-day" min="1" max="31" maxlength="2" pattern="[0-9]*" inputmode="numeric" autocomplete="bday-day" bind:value={day} data-pattern-mismatch="Enter a number for released before day">
    </div>
    <div class="ons-field">
      <label class="ons-label" for="before-date-month">Month</label>
      <input type="text" id="before-date-month" class="ons-input ons-input--text ons-input-type__input ons-input-number--w-2" name="before-month" min="1" max="12" maxlength="2" pattern="[0-9]*" inputmode="numeric" autocomplete="bday-month" bind:value={month} data-pattern-mismatch="Enter a number for released before month">
    </div>
    <div class="ons-field">
      <label class="ons-label" for="before-date-year">Year</label>
      <input type="text" id="before-date-year" class="ons-input ons-input--text ons-input-type__input ons-input-number--w-4" name="before-year" min="1000" max="3000" maxlength="4" pattern="[0-9]*" inputmode="numeric" autocomplete="bday-year" bind:value={year} data-value-missing="Enter the released before year" data-pattern-mismatch="Enter a number for released before year">
    </div>
  </div>
</fieldset>