<template>
    <div class="znwyy-main main" > 
        <el-scrollbar  wrap-class="el-picker-panel__content iscroll-warp">
            <div class="main-content" >
                <h2 class="sgintitle bordrtSet">通信管理</h2>
                <h2 class="sgintitle modeltitle">智能网语音</h2>
                <div>
                    <!-- 车联网集团智能语音列表先放这里 -->
                    <znwyy-x-menu></znwyy-x-menu>
                </div>
                    <h2 class="sgintitle modeltitle">基本信息</h2>
                    <jbxx></jbxx>
                    <div class="btn-div">
                        <el-button @click="hbmdBtnClick()">设置黑白名单</el-button>
                    </div>
                <div>
                    <!-- 基本信息下的信息列表放这里 -->
                </div>
                <h2 class="sgintitle modeltitle">用量信息</h2>
                <div>
                    <ylxx></ylxx>
                </div>
                <h2 class="sgintitle modeltitle">成员信息</h2>
                <div>
                    <!-- 成员信息下的信息列表放这里 -->
                    <znwyyTable :message="headData"></znwyyTable>
                </div>
            </div>
            <v-footer></v-footer>
        </el-scrollbar>
        <transition name="el-fade-in">
            <black-and-white-pop v-show="isShowBlackWhite"></black-and-white-pop>
        </transition>
    </div>
</template>

<script>
// 数据请求
import {getIntelligentVoiceMemberInfo, getEchartData, isSuccess} from '../api/index';
// 智能网语音列表
import znwyyTable from '../../biz_txgl/componet/table/znwyy_table';
// 脚试图
import vFooter from '../../common/component/footer/footerContact';
// 按钮列表
import znwyyXMenu from '../componet/xMenu/znwyyXMenu';
// 用量信息
import ylxx from '../componet/ylxx/index';
// 黑白名单弹窗
import blackAndWhitePop from '../componet/popView/blackAndWhitePop';
// 基本信息
import jbxx from '../componet/jbxx/znwyy-jbxx';

export default {
    name: 'txgl-znwyy',
    data() {
        return {
            hasNet: false,
            hasData: false,
            headData: [
                {item: 'MSISDN'},
                {item: 'ICCID'},
                {item: 'IMSI'},
                {item: '备注'},
                {item: '组织'},
                {item: '卡状态'},
                {item: '本月语音累计使用(分钟)'}
            ],
            tBodyData: [],
            isShowBlackWhite: false,
            groupId: '',
            pageNo: "1",
            pageSize: "10",
        }
    },
    components: {
        znwyyXMenu,
        ylxx,
        vFooter,
        znwyyTable,
        blackAndWhitePop,
        jbxx
    },
    created() {
        // 黑白名单弹窗
        this.$root.eventHub.$on('isShowBlackWhite',(target) => {
            this.isShowBlackWhite = target;
        });
         // 获取groupId
        this.$root.eventHub.$on('outGroupId',(target) => {
            /*
            * TODO
            * 请求页面初始化时默认table数据
            */
            this.groupId = target;
            let req = {
                pageNo: this.pageNo,
                pageSize: this.pageSize,
                groupId: target
            }
            this.getMumberInfo(req);
        });
        // 获取分页
        this.$root.eventHub.$on('mainfenyeData',(target) => {
            this.pageSize = target.tiaoshu;
            this.pageNo = target.yema;
            let req = {
                pageNo: this.pageNo,
                pageSize: this.pageSize,
                groupId: this.groupId
            }
            if (this.groupId != '') {
                this.getMumberInfo(req);
            }
        });
    },
    mounted() {
    },
    computed: {
    },
    methods: {
        getMumberInfo(req) {
            this.$root.eventHub.$emit('changeIntelligentVoiceMemberInfoDataShow', true);
            getIntelligentVoiceMemberInfo(req).then(response => {
                if(isSuccess(response)){
                    // 将数据发送给分页模块
                    this.$root.eventHub.$emit('toFenyeData', response.data.data);
                    if(response.data.data.list.length != 0) {
                        this.tBodyData = response.data.data.list;
                        this.hasNet = true;
                        this.hasData = true;
                        
                    } else {
                        this.hasNet = true;
                        this.hasData = false;
                    }
                    let tempData = {
                        hasNet: this.hasNet,
                        hasData: this.hasData,
                        data: this.tBodyData
                    }
                    this.$root.eventHub.$emit('changeIntelligentVoiceMemberInfoData', tempData);
                } else {
                    this.hasNet = false;
                    this.hasData = false;
                    let tempData = {
                        hasNet: this.hasNet,
                        hasData: this.hasData,
                        data: this.tBodyData
                    }
                    this.$root.eventHub.$emit('changeIntelligentVoiceMemberInfoData', tempData);
                }
               
            }).catch( error => {
                this.hasNet = false;
                this.hasData = false;
                    let tempData = {
                    hasNet: this.hasNet,
                    hasData: this.hasData,
                    data: this.tBodyData
                }
                this.$root.eventHub.$emit('changeIntelligentVoiceMemberInfoData', tempData);
            });
        },
        hbmdBtnClick() {
            this.isShowBlackWhite = true;
        },
        beforeDestroy() {
            this.$root.eventHub.$off('isShowBlackWhite');
            // 获取groupId
            this.$root.eventHub.$off('outGroupId');
            // 获取分页
            this.$root.eventHub.$off('mainfenyeData');
        }
    }
};
</script>

</script>

<style lang="scss">
.znwyy-main {
    .sgintitle {
        width: 100%;
        padding: 0.15rem 0;
        font-size: 0.24rem;
        text-align: left;
    }
    .modeltitle {
        font-size: 0.16rem;
        text-align: left;
        font-weight: 400;
    }
    .reminder {
        padding-top: 0;
        margin-top: 0;
        font-size: 0.12rem;
        text-align: left;
        font-weight: 400;
        color: #ff5f57;
    }
    .paddingset {
         padding: 0.15rem 0 0.26rem 0;
    }
    .paddingbotset {
         padding: 0 0 0.26rem 0;
    }
    .bordrtSet {
        border-bottom: 1px solid #e8ebf2;
    }
    .fixed-bottom {
        position: absolute;
        bottom: 0;
        left: 0;
        right: 0;
    }
    // 基本信息按钮CSS
    .btn-div {
        padding: 0.15rem 0.15rem;
        border: 1px solid #e8ebf2; 
        border-top: 0;
        text-align: right;
    }
    .el-button {
        width: 1.5rem;
    }
    .el-button--default {
        font-size: 0.14rem;
        background-color: #fff;
        border-color: #49a1ea;
        color: #49a1ea;
    }
}


</style>
