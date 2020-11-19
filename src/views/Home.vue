<template>
    <div class="home">
        <v-app id="inspire">
            <v-container class="grey lighten-5">
                <!--
                <v-row>
                    <v-col offset-md="3" md="6">
                        <v-alert type="warning">
                            Site still in construction.
                        </v-alert>
                    </v-col>
                </v-row>
                -->
                <v-row>
                    <v-col>
                        <h1> Ramjet Nodes </h1>
                        <div class="update-time">
                            <span class="utc">Updated: {{ dateString(timeUTC) }}</span>
                            <span class="d-none d-md-inline"><v-divider vertical style="display: inline;"></v-divider></span>
                            <time-ago class="ago d-block d-md-inline" :refresh="60" :datetime="timeUTC" locale="en" :key="timeUTC.getTime()" tooltip long></time-ago>
                        </div>
                    </v-col>
                </v-row>
                <v-row>
                    <v-col>
                        <v-simple-table class="fixed-table node-info">
                            <template v-slot:default>
                                <thead>
                                    <tr>
                                        <th class="text-center node">
                                            Node<br>(click for details)
                                        </th>
                                        <th class="text-center uptime">
                                            Up Time
                                        </th>
                                        <th class="text-center stakedtime">
                                            Staked Time
                                        </th>
                                        <th class="text-center last-sent">
                                            Last Sent<br>(secs ago)
                                        </th>
                                        <th class="text-center last-sent">
                                            Last Received<br>(secs ago)
                                        </th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr v-for="item in data" :key="item.idx">
                                        <td class="nodeID">
                                            <node :item="item"></node>
                                        </td>
                                        <td>
                                            <v-progress-linear :value="item[0].upTime" color="light-green" height="25">
                                                <template v-slot="{ value }">
                                                    <strong>{{ Math.ceil(value) }}%</strong>
                                                </template>
                                            </v-progress-linear>
                                        </td>
                                        <td>
                                            <v-progress-linear :value="item[0].upTime * item[1].stakedTime / 100" :buffer-value="item[0].stakedTime" color="blue-grey" height="25" stream>
                                                <template v-slot="{ value }">
                                                    <strong>{{ Math.ceil(value) }}%</strong>
                                                </template>
                                            </v-progress-linear>
                                        </td>
                                        <td>
                                            <template v-for="e in item">
                                                <span class="network-time" v-bind:key="e.provider"><span v-if="isDate(e.lastSent)">{{showLapse(new Date(e.lastSent), new Date(timeUTC))}}</span><span v-else>n/a</span><span class="comma">, </span></span>
                                            </template>
                                        </td>
                                        <td>
                                            <template v-for="e in item">
                                                <span class="network-time" v-bind:key="e.provider"><span v-if="isDate(e.lastReceived)">{{showLapse(new Date(e.lastReceived), new Date(timeUTC))}}</span><span v-else>n/a</span><span class="comma">, </span></span>
                                            </template>
                                        </td>
                                    </tr>
                                </tbody>
                            </template>
                        </v-simple-table>
                    </v-col>
                </v-row>
                <v-row>
                    <v-col>
                        <h1> Market Prices </h1>
                        <div class="update-time">
                            <span class="utc">Updated: {{ dateString(timePrice) }}</span>
                            <span class="d-none d-md-inline"><v-divider vertical style="display: inline;"></v-divider></span>
                            <time-ago class="ago d-block d-md-inline" :refresh="60" :datetime="timePrice" :key="timePrice.getTime()" locale="en" tooltip long></time-ago>
                        </div>
                    </v-col>
                </v-row>
                <v-row>
                    <v-col>
                        <v-simple-table class="fixed-table price-info">
                            <template v-slot:default>
                                <thead>
                                    <tr>
                                        <th class="text-center" v-for="header in prices.header" v-bind:key="header">
                                            {{ header }}
                                        </th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr v-for="entry in prices.entries" v-bind:key="entry[0]">
                                        <td v-for="(cell, idx) in entry.data" v-bind:key="idx">
                                            <span v-if="idx == 1 && entry.url">
                                                <a :href="entry.url" target="_blank">{{ cell }}</a>
                                            </span>
                                            <span v-else>
                                                {{ cell }}
                                            </span>
                                        </td>
                                    </tr>
                                </tbody>
                            </template>
                        </v-simple-table>
                    </v-col>
                </v-row>

            </v-container>
        </v-app>
    </div>
</template>

<script>
import node from './Node.vue'
import TimeAgo from 'vue2-timeago'
import dateFormat from 'dateformat'

export default {
    name: 'Home',
    data () {
        return {
            data: null,
            prices: {headers: []},
            timeUTC: new Date(),
            timeLocal: null,
            timePrice: new Date(),
        }
    },
    components: {
        node,
        TimeAgo,
    },
    mounted() {
        this.getStakerData();
        this.getPriceData();
    },
    methods: {
        dateString(date) {
            return dateFormat(date, "ddd mmm dd HH:MM:ss (Z)")
        },
        isDate(dateStr) {
            return !isNaN(new Date(dateStr).getDate())
        },
        showLapse(a, b) {
            return ((b - a) / 1000).toPrecision(2)
        },
        getStakerData() {
            this.$http.get('https://tedyin.com/archive/avax-stakers-api')
                .then(response => {
                    let rawData = response.data.results;
                    let data = [];
                    for (var i = 0; i < rawData[0].stakers.length; i++)
                    {
                        let item = [];
                        rawData.forEach(e => {
                            item.push({
                                provider: e.provider,
                                nodeID: e.stakers[i].nodeID,
                                status: e.stakers[i].status,
                                owner: e.stakers[i].owner,
                                amount: e.stakers[i].amount,
                                upTime: e.stakers[i].upTime,
                                stakedTime: e.stakers[i].stakedTime,
                                reward: e.stakers[i].reward,
                                lastSent: e.stakers[i].lastSent,
                                lastReceived: e.stakers[i].lastReceived,
                            });
                        });
                        data.push(item);
                    }
                    this.timeUTC = new Date(response.data.timeUTC);
                    this.timeLocal = response.data.timeLocal;
                    this.data = data;
                    setTimeout(this.getStakerData, 60000);
                })
                .catch(error => {
                    console.log(error);
                    setTimeout(this.getStakerData, 60000);
                })
        },
        getPriceData() {
            this.$http.get('https://tedyin.com/archive/avax-prices-api')
                .then(response => {
                    this.prices = {
                        header: response.data.header.slice(1),
                        entries: response.data.entries.map(entry => {
                            entry.data = entry.data.slice(1);
                            return entry
                        }),
                    };
                    this.timePrice = new Date(response.data.time);
                    setTimeout(this.getPriceData, 60000);
                })
                .catch(error => {
                    console.log(error);
                    setTimeout(this.getPriceData, 60000);
                })
        }
    },
}
</script>

<style>
.update-time {
    font-size: 1rem;
    font-family: monospace;
}
.update-time span {
    vertical-align: middle;
}
.update-time .utc {
    margin: 1ex;
}
.update-time .ago {
    margin: 1ex;
}
.nodeID {
    font-family: monospace;
}
.owner {
    font-family: monospace;
}
.v-btn {
    text-transform:none !important;
}
.network-time:last-child .comma {
    display: none;
}
.node-info .uptime {
    width: 10%;
    min-width: 100px;
}
.node-info .stakedtime {
    min-width: 100px;
}
.node-info .last-sent {
    width: 1%;
    min-width: 120px;
}
.node-info .node {
    width: 1%;
}
.fixed-table table > tbody > tr > td:nth-child(1), 
.fixed-table table > thead > tr > th:nth-child(1) {
    position: sticky !important; 
    position: -webkit-sticky !important; 
    border-top: 1px solid white;
    border-right: 1px solid black;
    left: 0; 
    z-index: 9998;
    background: white;
}
.fixed-table table > thead > tr > th:nth-child(1) {
    z-index: 9999;
}
</style>
