<template>
    <div class="home">
        <h1>Lowball</h1>
        <section v-if="trendData && trendData.length > 0">
        <trend
                :data="trendData"
                gradientDirection="top"
                :gradient="['#f94144', '#ffe74c', '#42b983']"
                :padding="10"
                :radius="6"
                :stroke-width="2"
                stroke-linecap="round"
                auto-draw
                smooth>
        </trend>

        <h1>Range: {{ minMaxPrice }} - {{ maxMaxPrice}} gwei</h1>
        <h2>Avg: {{ lowballAvg }} gwei</h2>
        </section>
        <section v-else>
            Fetching data...
        </section>
    </div>
</template>

<script>
  import axios from 'axios';
  import _ from 'lodash';

  export default {
    name: 'Home',
    data() {
      return {
        maxMaxPrice: null,
        minMaxPrice: null,
        trendData: [],
        polling: null
      };
    },
    computed: {
        lowballAvg() {
          return _.mean(this.trendData).toFixed(2);
        }
    },
    methods: {
      pollData() {
        this.polling = setInterval(() => {
          axios.get(
            'https://api.blocknative.com/gasprices/distribution',
            {
              headers: {
                'Authorization': `0a3cd861-402a-460f-af73-310a1cad1d33`
              }
            })
            .then(({data}) => {
              const lowball = _.last(data.topNDistribution.distribution)[0]

              this.trendData.push(lowball);
              if (!this.maxMaxPrice || lowball > this.maxMaxPrice) {
                this.maxMaxPrice = lowball;
              }
              if (!this.minMaxPrice || lowball < this.minMaxPrice) {
                this.minMaxPrice = lowball;
              }
            });
        }, 5000);
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
