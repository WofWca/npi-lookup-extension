<script>
  //@ts-check

  // I wrote these myself. Maybe it's wise to find a proper source.
  /**
   * @typedef {{
   *   country_code: string;
   *   country_name: string;
   *   address_purpose: string;
   *   address_type: string;
   *   address_1: string;
   *   city: string;
   *   state: string;
   *   postal_code: string;
   *   telephone_number: string;
   *   fax_number?: string;
   * }} Address
   */
  /**
   * @typedef {{
   *   organization_name: string;
   *   organizational_subpart: string;
   *   parent_organization_legal_business_name: string;
   *   enumeration_date: string;
   *   last_updated: string;
   *   certification_date: string;
   *   status: string;
   *   authorized_official_first_name: string;
   *   authorized_official_last_name: string;
   *   authorized_official_telephone_number: string;
   *   authorized_official_title_or_position: string;
   * }} BasicInfo
   */
  /**
   * @typedef {{
   *   code: string;
   *   taxonomy_group: string;
   *   desc: string;
   *   state: string | null;
   *   license: string | null;
   *   primary: boolean;
   * }} Taxonomy
   */
  /**
   * @typedef {{
   *   created_epoch: string;
   *   enumeration_type: string;
   *   last_updated_epoch: string;
   *   number: string;
   *   addresses: [Address, Address];
   *   practiceLocations: any[];
   *   basic: BasicInfo;
   *   taxonomies: Taxonomy[];
   *   identifiers: any[];
   *   endpoints: any[];
   *   other_names: any[];
   * }} Result
   */
  /**
   * @typedef {{
   *   result_count: number;
   *   results: Result[];
   * }} Response
   */

  let npiId = "1447496559";

  /** @type {Promise<Result | null | "neverFetched">}*/
  let resultP = Promise.resolve("neverFetched");

  async function onSubmit() {
    // Their `X-Frame-Options` denies this
    // frameSrc = `https://npiregistry.cms.hhs.gov/provider-view/${npiId}`;
    resultP = getResult();
  }

  async function getResult() {
    // https://npiregistry.cms.hhs.gov/api-page
    // FYI yes, the user _might_ inject other query parameters here.
    //
    // `host_permissions` in `manifest.json is required
    // for this origin to allow CORS.
    // The response doesn't set `Access-Control-Allow-Origin: *`.
    const res = await fetch(
      `https://npiregistry.cms.hhs.gov/api/?version=2.1&number=${npiId}`
    );
    const resData = /** @type {Response} */ (await res.json());
    if (resData.results.length <= 0) {
      return null;
    } else if (resData.results.length > 1) {
      console.warn("Expected just one result, got", resData.results.length);
    }

    return resData.results[0];
  }
</script>

<main>
  <form id="form" on:submit|preventDefault={onSubmit}>
    <label for="idInput">NPI number (10 digits)</label><br>
    <!-- https://npiregistry.cms.hhs.gov/demo-api
    The "number" field is exactly 10 digits-->
    <input
      type="search"
      id="idInput"
      name="idInput"
      minlength="10"
      maxlength="10"
      size="10"
      pattern="\d*"
      spellcheck="false"
      required
      bind:value={npiId}
    />
    <input type="submit" />
  </form>

  {#await resultP}
    <p>Fetching...</p>
  {:then result}
    <output form="form">
      <!-- svelte-ignore empty-block -->
      {#if result == null}
        <p>No results 🤷‍♀️</p>
      {:else if result === "neverFetched"}{:else}
        <!-- https://npiregistry.cms.hhs.gov/provider-view/1447496559
        is taken as a reference -->
        <table>
          <thead></thead>
          <tbody>
            <!-- TODO should probably also print unhandled fields -->
            {#each [
              "enumeration_type",
              "number",
            ] as propName}
              <tr>
                <td>{propName}</td>
                <td>{result[propName] ?? ''}</td>
              </tr>
            {/each}

            {#each [
              "organization_name",
              "organizational_subpart",
              "parent_organization_legal_business_name",
              "enumeration_date",
              "last_updated",
              "certification_date",
              "status",
              "authorized_official_first_name",
              "authorized_official_last_name",
              "authorized_official_telephone_number",
              "authorized_official_title_or_position",
            ] as propName}
              <tr>
                <td>{propName}</td>
                <td>{result.basic[propName] ?? ''}</td>
              </tr>
            {/each}
          </tbody>
        </table>

        <section>
          <h3>Primary Practice Location</h3>
          <table>
            <thead></thead>
            <tbody>
              {#each [
                "country_code",
                "country_name",
                "address_purpose",
                "address_type",
                "address_1",
                "city",
                "state",
                "postal_code",
                "telephone_number",
                "fax_number",
              ] as propName}
              <tr>
                <td>{propName}</td>
                <td>{result.addresses[0][propName] ?? ''}</td>
              </tr>
              {/each}
            </tbody>
          </table>
        </section>

        <section>
          <h3>Mailing Address</h3>
          <table>
            <thead></thead>
            <tbody>
              {#each [
                "country_code",
                "country_name",
                "address_purpose",
                "address_type",
                "address_1",
                "city",
                "state",
                "postal_code",
                "telephone_number",
                "fax_number",
              ] as propName}
              <tr>
                <td>{propName}</td>
                <td>{result.addresses[1][propName] ?? ''}</td>
              </tr>
              {/each}
            </tbody>
          </table>
        </section>

        <section>
          <h3>Taxonomies</h3>
          {#each result.taxonomies as taxonomy}
            <table>
              <thead></thead>
              <tbody>
                {#each [
                  "code",
                  "taxonomy_group",
                  "desc",
                  "state",
                  "license",
                  "primary",
                ] as propName}
                <tr>
                  <td>{propName}</td>
                  <td>{taxonomy[propName] ?? ''}</td>
                </tr>
                {/each}
              </tbody>
            </table>
          {/each}
        </section>
        <!-- {JSON.stringify(result)} -->
      {/if}
    </output>
  {:catch _error}
    <output form="form">Failed to fetch data 😖</output>
  {/await}
</main>

<style>
  td {
    border: 1px solid gray;
  }
</style>
