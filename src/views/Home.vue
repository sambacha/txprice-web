<template>
    <article class="container content" style="margin-bottom: 200px">
        <h1 class="has-text-centered is-size-1 has-text-primary" style="margin-top: 50px;">DeepGwei</h1>
        <hr/>
        <div class="columns">
            <div class="column">
                <line-chart :chart-data="datacollection" :options="options" v-if="totalTxs"></line-chart>
                <h3 v-else class="is-family-monospace has-text-grey-light">⏰ rendering graph...</h3>
            </div>
            <div class="column">
                <h2>Ethereum Transaction Pricing</h2>
                <p>
                   
                </p>
                <p>
                    
                </p>

                <h3 class="is-family-monospace has-text-grey-light" v-if="totalTxs">{{ totalTxs }} pending transactions</h3>
                <h3 v-else class="is-family-monospace has-text-grey-light">accessing mempool...</h3>

                <div class="notification is-light" v-if="winning">
                    <span class="is-size-3">⛽️</span> Suggested pricing<span class="has-text-weight-semibold">{{ winning.key }} gwei</span> would be accepted within the next 5 blocks
                </div>

                <b-field label="Transaction Inclusion Preference" message="Adjust based on transaction settlement time" style="margin-top: 40px">
                    <b-slider v-model="modulo" :min="5" :max="50" :step="5"></b-slider>
                </b-field>

                <b-field label="Reset" message="purge cache and establish a new connection">
                    <button @click="reset" class="button is-primary is-outlined">Reset</button>
                </b-field>

                <blockquote style="margin-top: 40px">
                  Powered by<a href="https://blocknative.com" target="_blank">Blocknative</a>'s the <a href="https://docs.blocknative.com/gas-platform" target="_blank">Gas Distribution feed</a>. This feed provides the distribution of the top gas prices in the mempool right now.
                </blockquote>
            </div>
        </div>


    </article>
</template>

<script>
  import LineChart from '../line-chart';
  import axios from 'axios';
  import _ from 'lodash';

  export default {
    components: {
      LineChart
    },
    data() {
      return {
        rawData: [],
        processedData: {},
        processedDataRounded: {},
        labelSet: [],
        dataSet: [],
        datacollection: {},
        winning: null,
        totalTxs: null,
        modulo: 25,
        options:  {
          scales: {
          xAxes: [ {
            display: true,
            scaleLabel: {
              display: true,
              labelString: 'Gas price in Gwei'
            },
          } ],
            yAxes: [ {
            display: true,
            scaleLabel: {
              display: true,
              labelString: '# of transactions'
            }
          } ]
        }
      },
      };
    },
    methods: {
      reset() {
        this.rawData = [];
        console.log('reset');
      },
      pollData() {
        this.polling = setInterval(() => {
          axios.get(
            'https://api.blocknative.com/gasprices/distribution',
            {
              headers: {
                'Authorization': `6f48615a-3c92-4adb-b593-69f10315f3ab`
              }
            })
            .then(({data}) => {

              this.rawData = this.rawData.concat(data.topNDistribution.distribution);

              this.processedData = _.reduce(this.rawData, (obj, rd) => {
                  obj[`${rd[0]}`] = obj[`${rd[0]}`] ? obj[`${rd[0]}`] + rd[1] : rd[1];
                  return obj;
                },
                {}
              );

              this.processedDataRounded = _.reduce(this.processedData, (obj, value, key) => {
                const newKey = parseInt(parseInt(key) / this.modulo)
                obj[`${newKey * this.modulo}`] = obj[`${newKey * this.modulo}`] ? obj[`${newKey * this.modulo}`] + value : value;
                return obj;
              }, {});

              this.totalTxs = _.reduce(this.processedData, (sum, value, key) => sum + value, 0)

              this.processedDataRounded = _.pickBy(this.processedDataRounded, (value, key) => value > 20);

              this.winning = _.last(_.orderBy(Object.keys(this.processedDataRounded).map(key => ({ key, value: this.processedDataRounded[key] })), 'value'));

              this.labelSet = _.keys(this.processedDataRounded);
              this.dataSet = _.values(this.processedDataRounded);

              this.datacollection = {
                labels: this.labelSet,
                datasets: [
                  {
                    label: `top gas prices in the mempool`,
                    backgroundColor: '#2188FF',
                    data: this.dataSet,
                    pointRadius: 0
                  }
                ]
              };
            });
        }, 2000);
      },
    },
    beforeDestroy() {
      clearInterval(this.polling);
    },
    created() {
      this.pollData();
    }
  };
</script>

<style>
    .small {
        max-width: 600px;
        margin: 150px auto;
    }
</style>