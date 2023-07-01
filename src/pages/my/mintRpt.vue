<!--Mint PRT-->
<script setup>
import { Icon } from '@iconify/vue';
import { ref, getCurrentInstance } from 'vue'
import { onLoad, onPullDownRefresh } from '@dcloudio/uni-app';
import { useUserStore } from "@/stores/userStore";
const userStore = useUserStore();
const { proxy } = getCurrentInstance();

const rptBalance = ref(0) //RPT余额
const rptSupply = ref(0) //新RPT流通
const mintTotal = ref(0) //mint统计
const mintState = ref(false) //mint状态
const btnLoading = ref(false) //按钮状态
const contractId = 'ct_2U1usf3A8ZNUcZLkZe5rEoBTxk7eJvk9fcbRDNqmRiwXCHAYN'

onLoad ( () => {
    getSupply();
    getBalance();
});

//下拉刷新
onPullDownRefresh ( () => {
    getSupply();
    getBalance();
    setTimeout(() => {
        uni.stopPullDownRefresh();
    }, 500);
});

//开始
const start = () => {
    if (proxy.validThirdPartySource()) {
        proxy.uShowToast(
            proxy.$t('index.thirdPartyNotOpen')
        );
        return false;
    };
    if ( rptBalance.value > 0 ) {
        mintState.value = true;
        btnLoading.value = true;
        mint();
    }
}
//Mint
const mint = () => {
    if (mintState.value === false || btnLoading.value === false) {
        mintState.value = false;
        btnLoading.value = false;
        return false;
    }
    proxy.contractTransfer(contractId, userStore.token, 1).then(()=>{
        getSupply();
        getBalance();
        mintTotal.value = mintTotal.value + 1
        mint();
    }).catch(() => {
        mintState.value = false;
        btnLoading.value = false;
    });
}
//停止
const stopMint = () => {
    mintState.value = false;
    btnLoading.value = false;
    proxy.uShowToast( 'Stop Mint...');
}
//返回首页
const goHome = () => {
    stopMint();
    proxy.reLaunchUrl('index');
}
//获取余额
const getBalance = () => {
    proxy.getTokenBalance(
        contractId,
        userStore.token
    ).then((res) => {
        rptBalance.value = proxy.balanceFormat(res, 2);
    });;
}
//获取总流通
const getSupply = () => {
    proxy.getTokenSupply( contractId ).then((res) => {
        rptSupply.value = proxy.balanceFormat(res, 0)
    });
}
//转换为会计格式
const toThousands = (num) => {
    num = (num || 0).toString();
    var decimal;
    if (num.lastIndexOf(".") != -1) {
        decimal = num.split(".")[1];
    }
    num = num.split(".")[0].replace(/(\d)(?=(?:\d{3})+$)/g, "$1,");
    if (decimal) {
        num += "." + decimal;
    }
    return num;
}
</script>

<template>
    <view class="m-token">
        <view class="icon-list" v-show="!validThirdPartySource()">
        <view :style="{height:`${statusBarHeight}px`}"></view>
            <Icon
                icon="octicon:home-16"
                color="#fff"
                class="mr-30"
                width="18"
                @click="goHome()"
            />
        </view>
        <view class="title">Mint RPT<br />Regenerates Points Token</view>
        <view class="start">
            <Icon
                class="trophy"
                icon="bi:sign-stop"
                width="28"
                color="#f04a82"
                @click="stopMint"
            />
            <view class="migrate">
                <view class="top">
                    <view class="desc">Total Supply</view>
                    <view class="earning">
                        <view class="amount">
                            {{ toThousands(rptSupply) }}
                        </view>
                    </view>
                </view>
                <view class="bottom">
                    <view class="migrate-num">
                        <view class="desc">Mint</view>
                        <u-gap :height="10"></u-gap>
                        <view class="num">
                            {{ toThousands(mintTotal) }}.00
                        </view>
                    </view>
                    <view class="migrate-total">
                        <view class="desc">Balance</view>
                        <u-gap :height="10"></u-gap>
                        <view class="num">
                            <u-count-to
                                :decimals="2"
                                :font-size="42"
                                :bold="true"
                                :start-val="rptBalance-1"
                                :end-val="rptBalance"
                            ></u-count-to>
                        </view>
                    </view>
                </view>
                <u-button
                    shape="circle"
                    type="primary"
                    @click="start()"
                    :loading="btnLoading"
                    >
                    Start Mint
                </u-button>
            </view>
        </view>
        <view class="rule">
            <view class="h3">Instructions</view>
            <u-gap :height="10"></u-gap>
            RPT: <br />
                    Regenerates Points Token<br />
            <u-gap :height="10"></u-gap>
            Project:<br />
                    No project party, Community anonymous deployment<br />
            <u-gap :height="10"></u-gap>
            Feature:<br />
                    Each transfer is automatically minted a little extra<br />
            <u-gap :height="10"></u-gap>
            Max supply:<br />
                    21 Million, with halving mechanism, same as BTC.<br />
            <u-gap :height="10"></u-gap>
            Participation: <br />
                    Unlimited regenerative casting with any quantity<br />
            <u-gap :height="10"></u-gap>
            Supplementary: <br />
                    WeTrue does not participate in the project and only facilitates casting
        </view>
    </view>
</template>

<style lang="scss" scoped>
page {
    background-color: #e6e6e6;
}
.m-token {
    background: url("@/static/blue_back_bg.png") no-repeat;
    background-size: 100%;
    display: flex;
    justify-content: center;
    flex-flow: wrap;
    position: relative;
    .icon-list {
        position: absolute;
        right: 0rpx;
        top: 30rpx;
    }
    & > .title {
        position: absolute;
        top: 150rpx;
        left: 10%;
        font-size: 46rpx;
        font-weight: 600;
        color: #fff;
        line-height: 70rpx;
    }
    .start {
        margin-top: 400rpx;
        background: #fff;
        width: 80%;
        min-height: 400rpx;
        border-radius: 30rpx;
        padding: 40rpx;
        box-sizing: border-box;
        position:relative;
        .trophy {
            position: absolute;
            right: 30rpx;
            top: 30rpx;
        }
        .title {
            font-size: 36rpx;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        .migrate {
            .desc {
                color: #666;
            }
            .top {
                border-bottom: 1px solid #eee;
                .earning {
                    display: flex;
                    justify-content: space-between;
                    align-items: center;
                    margin: 10rpx 0 30rpx 0;
                    .amount {
                        color: #f04a82;
                        font-size: 42rpx;
                        font-weight: bold;
                    }
                }
            }
            .bottom {
                padding: 30rpx 0;
                display: flex;
                justify-content: space-between;
                align-items: center;
                .num {
                    color: #000;
                    font-size: 42rpx;
                    font-weight: 600;
                }
            }
        }
    }
    .rule {
        color: #000;
        width: 80%;
        padding: 50rpx 0;
        line-height: 40rpx;
        font-size: 20rpx;
        .h3 {
            color: #000;
            font-size: 28rpx;
            font-weight: 600;
        }
    }
}
</style>
