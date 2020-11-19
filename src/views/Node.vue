<template>
    <div>
    <v-btn
        class="ma-2"
        color="normal"
        @click="toggle()"
        case="user"
        >
        <span :set="nodeID = item[0].nodeID">
        {{ nodeID.substring(0, 4) + '...' + nodeID.substring(nodeID.length - 4, nodeID.length) }}
        </span>
        <v-divider class="btn-divider"></v-divider>
        <template v-for="e in item">
            <v-badge v-bind:key="e.provider" inline v-bind:color="e.status == 'ok' ? 'light-green' : 'red'" dot></v-badge>
        </template>
    </v-btn>
    <v-expand-transition>
        <div v-show="isActive" class="item-desc">
            <table class="node-detail">
                <tr class="owner"><td>ID:</td><td>NodeID-{{ item[0].nodeID }}</td></tr>
                <tr class="owner"><td>Owner:</td><td>{{ item[0].owner }}</td></tr>
                <tr class="owner"><td>Amount:</td><td><span v-if="item[0].status == 'ok'">{{ item[0].amount }}</span><span v-else>n/a</span></td></tr>
                <tr class="owner"><td>Reward:</td><td><span v-if="item[0].status == 'ok'">{{ item[0].reward }}</span><span v-else>n/a</span></td></tr>
                <template v-for="e in item">
                    <tr v-bind:key="e.provider">
                        <td>{{ e.provider }}:</td>
                        <td>status={{ e.status }}
                            upTime=<span v-if="!isNaN(e.upTime)">{{ e.upTime.toPrecision(4) + '%' }}</span><span v-else>n/a</span>
                            stakedTime=<span v-if="!isNaN(e.stakedTime)">{{ e.stakedTime.toPrecision(4) + '%' }},{{ (e.upTime * e.stakedTime / 100).toPrecision(4) + '%' }}</span><span v-else>n/a</span></td>
                    </tr>
                </template>
            </table>
        </div>
    </v-expand-transition>
    </div>
</template>

<script>
export default {
    props: {
        item: {
            type: Array,
            required: true,
        }
    },
    methods: {
        toggle() {
            this.isActive = !this.isActive;
        }
    },
    data() {
        return {
            isActive: false
        }
    },
}
</script>

<style>
.btn-divider {
    margin: 1ex;
}
.node-detail {
    min-width: 60ex;
    margin: auto;
}
.node-detail tr td {
    text-align: right;
    vertical-align: top;
}
.node-detail tr td + td{
    text-align: left;
}
</style>
