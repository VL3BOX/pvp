<template>
    <div class="m-pvp-sandbox__content" ref="sandboxMap">
        <sandboxSearch :servers="servers" @sandboxChange="onSandbox" />
        <div class="m-sandbox-map">
            <sandboxMaps :maps="sandMaps" :camp="camp" :route="route" @mapClick="mapClick"> </sandboxMaps>
            <sandboxLog :item="itemLog" v-if="showLog" />
        </div>
        <div class="m-mask" v-if="showLog" @click="closeLog"></div>
    </div>
</template>
<script>
import { onClickOutside } from "@vueuse/core";
import sandboxSearch from "@/components/sandbox/SandboxSearch.vue";
import sandboxMaps from "@/components/sandbox/SandboxMaps.vue";
import sandboxLog from "@/components/sandbox/SandboxLog.vue";
import servers_std from "@jx3box/jx3box-data/data/server/server_std.json";
import { getCamplist, getCampLog } from "@/service/sandbox";
import { useI18n } from "vue-i18n";

export default {
    name: "SandBox",
    components: {
        sandboxSearch,
        sandboxMaps,
        sandboxLog,
    },
    data: function () {
        return {
            sandMaps: {},
            servers: servers_std.map((name) => this.t(name)),
            itemLog: "",
            showLog: false,
            server: this.t(servers_std[0]) || "",
        };
    },
    computed: {
        parms: function () {
            return {
                server: this.server,
                camp: this.camp,
            };
        },
    },
    mounted() {
        onClickOutside(this.$refs.sandboxMap, () => {
            if (this.showLog) {
                this.showLog = false;
            }
        });
    },
    methods: {
        //搜索更改 parms的条件
        onSandbox(data) {
            let { server, camp, route } = data;
            this.server = server;
            this.camp = camp;
            this.route = route;
            this.getSandbox();
            this.$emit("sandboxChangeKey", server);
        },
        //获取沙盘数据 含沙盘攻防路线
        getSandbox() {
            getCamplist(this.parms).then((res) => {
                this.sandMaps = { list: res.data.castles };
            });
        },
        // 点击展示日志
        mapClick(item) {
            getCampLog(this.id, item.id).then((res) => {
                item.list = res.data.data;
                this.itemLog = item;
                this.showLog = true;
            });
        },
        closeLog() {
            this.showLog = false;
        },
    },

    created: function () {
        this.getSandbox();
    },
    setup() {
        const { t } = useI18n();
        return {
            t,
        };
    },
};
</script>
<style lang="less">
// @import "~@/assets/css/sandbox/sandbox.less";
</style>
