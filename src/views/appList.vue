<template>
    <!-- <div ref="wrap" style="height: 1000px;">
        <navbarLight :list="titleList" @navbarClick="handleNavbarClick" v-model="active" v-if='!isError' :color='themeColor'></navbarLight>
        <bui-dropload @loading="loading" @refresh="refresh" :hasRefresh='false' :hasLoading='false' v-if='!isError'>
            <cell ref="item" v-for="(item, index) in sortList" :key="item.id">
                <div @appear="(e)=>{handleAppAppear(e,index)}" @disappear="(e)=>{handleAppDisappear(e,index)}">
                    <subtitle :title="item.name" margin-top="0" :color='themeColor'></subtitle>
                </div>
                <div class="menu__listItem">
                    <div class="menu__item" @click="myAllpyEvent(item)" v-for="item in item.data" :key="item.id">
                        <bui-image class="menu__image" @click="myAllpyEvent(item)" width="60px" height="60px" :src="item.icon" resize="contain"></bui-image>
                        <text class="menu__title" @click="myAllpyEvent(item)">{{item.name}}</text>
                    </div>
                </div>
            </cell>
            <cell style="height:400px;background-color: #fff;"></cell>
        </bui-dropload>
        <div class="no-content flex-ac flex-jc" v-if='isShow && isError'>
            <div class="flex-dr">
                <bui-image src="/image/nodata.png" width="20wx" height="20wx"></bui-image>
                <text class="f26 c51 fw4 pl15 center-height">{{i18n.ErrorLoadData}}</text>
            </div>
        </div>
    </div> -->
    <div ref="wrap" class="main">
        <div ref="item" v-for="(item, index) in sortList" :key="item.id">
            <div @appear="(e)=>{handleAppAppear(e,index)}" @disappear="(e)=>{handleAppDisappear(e,index)}" v-if='item.data.length != 0'>
                <subtitle :title="item.name" margin-top="0" :color='themeColor'></subtitle>
            </div>
            <div class="menu__listItem" v-if='item.data.length != 0'>
                <div class="menu__item" @click="myAllpyEvent(item)" v-for="item in item.data" :key="item.id">
                    <bui-image class="menu__image" @click="myAllpyEvent(item)" width="60px" height="60px" :src="item.icon" resize="contain"></bui-image>
                    <text class="menu__title" @click="myAllpyEvent(item)">{{item.name}}</text>
                </div>
            </div>
        </div>
        <div class="no-content flex-ac flex-jc" v-if='isShow && isError'>
            <div class="flex-dr">
                <bui-image src="/image/nodata.png" width="20wx" height="20wx"></bui-image>
                <text class="f26 c51 fw4 pl15 center-height">{{i18n.ErrorLoadData}}</text>
            </div>
        </div>
    </div>
</template>

<script>
const link = weex.requireModule("LinkModule");
const linkapi = require("linkapi");
const dom = weex.requireModule("dom");
const storage = weex.requireModule('storage');
var storeFileIdUiDownload = '/store/getFile?fileId=${id}';
import subtitle from '../components/subtitle.vue';
import navbarLight from '../components/navbarLight.vue';
export default {
    data() {
        return {
            type: '',
            titleList: [],
            list: [],
            active: 0,
            stopTriggerByScroll: false,
            isShow: false,
            isError: true,
            i18n: '',
            channel: new BroadcastChannel("WidgetsMessage"),
            themeColor: ''
        };
    },
    components: {
        subtitle,
        navbarLight
    },
    computed: {
        sortList() {
            return this.transformData(this.titleList, this.list)
        },
    },
    created() {
        this.$fixViewport();
        linkapi.getLanguage(res => {
            this.i18n = this.$window[res];
            this.refreshTextMap = {
                start: this.$window[res].start,
                move: this.$window[res].move,
                success: this.$window[res].success,
                fail: this.$window[res].fail
            }
        });
        var type;
        if (this.$isIPad()) {
            type = 6
        } else if (this.$isAndroid()) {
            type = 1
        } else if (this.$isIPhone()) {
            type = 2
        }
        this.type = type
        linkapi.getThemeColor(res => {
            this.themeColor = res.background_color;
        })
    },
    mounted() {
        var that = this
        this.channel.onmessage = event => {
            if (event.data.action === "RefreshData") {
                this.getData();
            }
        };
        this.getStorage(function () {
            that.getData()
        })
    },
    methods: {
        getStorage(callback) {
            storage.getItem('appListLocalData', res => {
                if (res.result == 'success') {
                    var data = JSON.parse(res.data)
                    this.isShow = true
                    this.isError = false
                    this.titleList = data[0] || [];
                    this.list = data[1] || [];
                    this.stopTriggerByScroll = true;
                    setTimeout(() => {
                        this.stopTriggerByScroll = false;
                    }, 300)
                    this.broadcastWidgetHeight()
                } else {
                    callback()
                }
            })
        },
        getData() {
            this.getServerConfig((params) => {
                Promise.all([this.fetchCategory(params.appApiUri), this.fetchApp(params.appApiUri)]).then(arr => {
                    this.isShow = true
                    this.isError = false
                    this.titleList = arr[0].data || [];
                    this.list = arr[1].data || [];
                    for (let i = 0; i < this.list.length; i++) {
                        var item = this.list[i];
                        var icon = item.icon ? item.icon.split('||')[0] : '/image/ic_service_default.png'
                        item.icon = icon != '' ? params.uamUri + '/ui/upload?action=download&filepath=' + icon : ''
                    }
                    this.stopTriggerByScroll = true;
                    setTimeout(() => {
                        this.stopTriggerByScroll = false;
                    }, 300)
                    let data = [this.titleList, this.list]
                    this.broadcastWidgetHeight()
                    storage.setItem('appListLocalData', JSON.stringify(data))
                }).catch(e => {
                    this.error(e)
                })
            })
        },
        fetchCategory(url) {
            return linkapi.get({
                url: url + "/escategory/with/escount",
                data: {
                    terminalType: this.type
                }
            })
        },
        fetchApp(url) {
            return linkapi.get({
                // 获取授权的全部应用
                url: url + "/v2/es/authorized",
                data: {
                    terminalType: this.type
                }
            })
        },
        getServerConfig(callback) {
            link.getServerConfigs([], params => {
                callback && callback(params)
            }, err => {
                this.error();
            });
        },
        error() {
            this.isError = true
            this.isShow = true
        },
        refresh(next) {
            setTimeout(() => {
                next && next();
            }, 200)
        },
        transformData(menu, data) {
            return menu.map(item => {
                // 初始化一个存放应用的数组
                item.data = item.data ? item.data : [];
                // 从所有应用中过滤当前菜单的应用
                let list = data.filter(inner => {
                    if (item.name == inner.categoryName) {
                        return inner;
                    }
                })
                // 将过滤出来的应用设置在当前应用的列表上
                if(list.length != 0){
                }
                item.data = item.data.concat(list || [])
                return item;
            })
        },
        handleNavbarClick({ index, item }) {
            this.stopTriggerByScroll = true;
            const el = this.$refs.item[index] || null;
            if (!el) {
                return
            }
            this.$nextTick(() => {
                dom.scrollToElement(el, {
                    offset: 0,
                    animated: false,
                })
                setTimeout(() => {
                    this.stopTriggerByScroll = false;
                }, 300)

            })
        },
        handleAppAppear(e, index) {
            if (this.stopTriggerByScroll) {
                return;
            }
            if (e.direction === 'down') {
                this.active = index;
            }
        },
        handleAppDisappear(e, index) {
            if (this.stopTriggerByScroll) {
                return;
            }
            let max = this.sortList.length - 1;
            if (e.direction === 'up') {
                this.active = Math.min(index, max);
            }
        },
        // 我的应用
        myAllpyEvent(item) {
            var appVersion = weex.config.env.appVersion
            try {
                var appInfo = link.getVersionInfo(
                    [],
                    res => {
                        appVersion = res.versionName
                    }
                )
                if (weex.config.env.platform == "iOS") {
                    this.heightEditionRunApp(item)
                } else {
                    if (appVersion.split('.')[0] < 4 || (appVersion.split('.')[0] == 4 && appVersion.split('.')[1] < 4)) {
                        this.lowEditionRunApp(item.actionParams, more)
                    } else {
                        this.heightEditionRunApp(item)
                    }
                }
            } catch (error) {
                this.lowEditionRunApp(item.actionParams)
            }
        },
        lowEditionRunApp(actionParams, more) {
            actionParams && link.launchLinkService([actionParams], (res) => { }, (err) => { });
        },
        heightEditionRunApp(item) {
            var that = this
            this.lowEditionRunApp(item.actionParams)
            link.addAnalysisLog([{
                moduleCategory: "应用",
                moduleName: "应用操作",
                eventType: "启动应用",
                eventEntry: null,
                eventParams: null,
                targetId: item.id,
                targetName: item.name
            }], () => { }, () => { })
        },
        broadcastWidgetHeight() {
            let _params = this.$getPageParams();
            // 防止高度通知失败
            setTimeout(() => {
                this.getComponentRect(_params)
            }, 200)
            setTimeout(() => {
                this.getComponentRect(_params)
            }, 1200)
        },
        getComponentRect(_params) {
            dom.getComponentRect(this.$refs.wrap, (ret) => {
                this.channel.postMessage({
                    widgetHeight: ret.size.height,
                    id: _params.id
                });
            });
        }
    }
};
</script>

<style lang="css" src="../css/common.css"></style>
<style>
.main {
    padding-bottom: 10px;
}
.menu__listItem {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    padding-bottom: 5px;
    background-color: #fff;
}
.menu__item {
    width: 182px;
    display: flex;
    align-items: center;
    margin-top: 30px;
}
.menu__image {
}
.menu__title {
    flex: 1;
    font-size: 24px;
    color: #000;
    margin-top: 15px;
    text-overflow: ellipsis;
    lines: 1;
}
.no-content {
    height: 200wx;
}
</style>