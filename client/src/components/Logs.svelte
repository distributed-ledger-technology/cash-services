<!-- http://localhost:3001/getAccountInfo/apiKey/GCNuPXHiTsX5FTEDhV -->
<script>
  import { SortService, Direction } from "../deps.ts";
  import { onMount } from "svelte";
  // import Chart from "./Chart.svelte";

  export let accountInfo;
  export let logs = [];

  export let sentiment = 0;

  function getDataSourceURL() {
    if (window.location == "http://localhost:3027/") {
      // for maintenance

      return `http://localhost:3001/getLogs/apiKey/${accountInfo.apiKey}`;
    }

    return `https://openforce.de/getLogs/apiKey/${accountInfo.apiKey}`;
  }

  function getSentiment() {
    if (accountInfo.longPositionSize > accountInfo.shortPositionSize) {
      if (
        accountInfo.longPositionPNLInPercent >
        accountInfo.shortPositionPNLInPercent
      )
        return 2;

      if (
        accountInfo.longPositionPNLInPercent > 0 &&
        accountInfo.shortPositionPNLInPercent > 0
      )
        return 1;

      if (
        accountInfo.shortPositionPNLInPercent >
        accountInfo.longPositionPNLInPercent
      )
        return -1;
    } else if (accountInfo.shortPositionSize > accountInfo.longPositionSize) {
      if (
        accountInfo.shortPositionPNLInPercent >
        accountInfo.longPositionPNLInPercent
      )
        return 2;

      if (
        accountInfo.longPositionPNLInPercent > 0 &&
        accountInfo.shortPositionPNLInPercent > 0
      )
        return 1;

      if (
        accountInfo.longPositionPNLInPercent >
        accountInfo.shortPositionPNLInPercent
      )
        return -1;
    }

    return 0;
  }

  // async function allDeals() {
  //   all = true;
  //   displayedDeals = deals;
  // }

  async function getLogs() {
    try {
      const url = getDataSourceURL();
      logs = await (await fetch(url)).json();

      const sortOptions = [
        { fieldName: "utcTime", direction: Direction.ASCENDING },
      ];

      logs = SortService.sort(logs, sortOptions);
    } catch (error) {
      console.log(error.message);
      alert(`I could not get any data for api key ${accountInfo.apiKey}`);
    }
  }

  onMount(async () => {
    await getLogs();
    setInterval(async () => {
      if (accountInfo.apiKey !== "") {
        sentiment = getSentiment();
        console.log(sentiment);
        await getLogs();
      }
    }, 4 * 1000);
  });
  // import InputField from './InputField.svelte';
</script>

<!-- <Chart /> -->

{#if logs.length > 0}
  <div id="logList">
    <h2>Last {logs.length} Log Entries</h2>

    <p><br /></p>
    <table
      class="terminalStyle"
      class:backgroundorangered={sentiment === -1}
      class:backgroundBlack={sentiment === 0}
      class:backgroundlightskyblue={sentiment === 1}
      class:backgroundgreen={sentiment === 2}
    >
      {#each logs as log}
        <tr>
          <td>{log.message}</td>
        </tr>
      {/each}
    </table>
  </div>
{/if}

<style>
  table {
    background-color: violet;
    font-family: arial, sans-serif;
    border-collapse: collapse;
    width: 100%;
  }

  td,
  th {
    /* border: 1px solid #dddddd; */
    text-align: center;
    padding: 8px;
  }

  /* tr:nth-child(even) {
    background-color: black;
    background-color: #dddddd;
  } */

  .backgroundBlack {
    background-color: black;
  }

  .backgroundorangered {
    background-color: orangered;
  }

  .backgroundlightskyblue {
    /* background-color: turquoise; */
    background-color: lightskyblue;
  }

  .backgroundgreen {
    /* background-color: turquoise; */
    background-color: green;
  }

  .terminalStyle {
    color: white;
  }
</style>
