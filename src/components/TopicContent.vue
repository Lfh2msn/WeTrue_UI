<script setup>
import HeadImg from "@/components/HeadImg";
import mpHtml from "mp-html/dist/uni-app/components/mp-html/mp-html";
import RewardRecord from "@/components/RewardRecord";
import Name from "@/components/Name";
import TopicMore from "@/components/TopicMore";
import Backend from "@/util/backend";
import { Icon } from '@iconify/vue';
import { ref, getCurrentInstance, watch, nextTick } from 'vue';
import { mixinUtils } from'@/mixins/mixinUtils'
import { useRouter } from "vue-router"
import { useUserStore } from "@/stores/userStore";
const userStore = useUserStore();
const { proxy } = getCurrentInstance();

const router = useRouter()

const props = defineProps({
    postInfo: {
        type: Object,
        default: () => {},
    },
})
const root = ref(null) //控制打赏记录弹层
const rewardRecordShow = ref(false) //控制打赏记录弹层

//暴露方法名"receiveWeTrueMessage"
window["receiveWeTrueMessage"] = async (res) => {
    if (res.code == 200) {
        proxy.postHashToWeTrue(res, true).then((res) => {
            releaseCallback(res);
        });
    }
};


watch(
	() => props.postInfo,
	(val) => {
        nextTick(() => {
            const topicArr =
                document.getElementsByClassName("topic-text");
            if (topicArr.length > 0) {
                for (let i = 0; i < topicArr.length; i++) {
                    topicArr[i].addEventListener(
                        "click",
                        (e) => {
                            let text = topicArr[i].innerText;
                            proxy.goUrl(
                                "/pages/index/topic?keyword=" + text
                            );
                            e.stopPropagation();
                        },
                        true
                    );
                }
            }
            const mentionsArr =
                document.getElementsByClassName("mentions-text");
            if (mentionsArr.length > 0) {
                for (let i = 0; i < mentionsArr.length; i++) {
                    mentionsArr[i].addEventListener(
                        "click",
                        (e) => {
                            let text = mentionsArr[i].innerText;
                            Backend.nodeApiGetAddressByNames(
                                text.slice(1)
                            ).then((res) => {
                                proxy.goUrl(
                                    "/pages/my/userInfo?userAddress=" +
                                        res
                                );
                            });
                            e.stopPropagation();
                        },
                        true
                    );
                }
            }
        });
	}
)

//是否收藏
const star = () => {
    let payload = {
        action: props.postInfo.isStar,
        content: props.postInfo.hash,
    };
    proxy.wetrueSend("star", payload).then((res) => {
        releaseCallback(res);
    });
}
const releaseCallback = (res) => {
    if (res.code == 200) {
        props.postInfo.isStar = res.data.isStar;
        props.postInfo.star = res.data.star;
    }
    proxy.uHideLoading();
}
const copy = () => {
    let text = root.value.getText();
    proxy.copyContent(text);
}
//分享
const share = () => {
    let shareUrl = userStore.configInfo.frontEndUrl + router.currentRoute.value.fullPath
    let text = root.value.getText();
    let shareTextLength = "";
    if (text.length > 100) {
        shareTextLength = "...";
    }
    let shareText =
        "WeTrue:\n" +
        shareUrl +
        "\n" +
        text.slice(0, 100) +
        shareTextLength;
        proxy.copyContent(shareText, "#share");
}
</script>

<template>
    <view class="forum">
        <view class="forum-item">
            <view class="focus" v-show="postInfo.isFocus">
                <Icon
                    icon="fa:heart"
                    width="12"
                    class="star"
                    color="#fff"
                />
            </view>
            <view class="user-area">
                <view class="head-box">
                    <HeadImg
                        :userInfo="postInfo.users"
                        :isLink="true"
                        width="70rpx"
                        height="70rpx"
                    ></HeadImg>
                </view>
                <view class="user-info">
                    <view class="user">
                        <Name :userInfo="postInfo.users"></Name>
                        <TopicMore
                            :topicInfo="postInfo"
                            class="mr-10"
                        ></TopicMore>
                    </view>
                    <view class="time">
                        <text class="mr-20">
                            {{ $moment(postInfo.utcTime).format("yyyy-MM-DD HH:mm") }}
                        </text>
                        <text class="mr-20">
                            {{ $t('index.source') + postInfo.source }}
                        </text>
                        
                    </view>
                </view>
            </view>
            <view class="main-content">
                <view class="text-content">
                    <mp-html
                        :content="postInfo.payload"
                        :selectable="true"
                        ref="root"
                    />
                </view>
                <view
                    class="img-list"
                    v-for="(item, index) in postInfo.mediaList"
                    :key="index"
                >
                    <u-image
                        height="640rpx"
                        mode = "heightFix"
                        :lazy-load="true"
                        :src="mixinUtils.ipfsUrltoCid(item.image.ipfs)"
                        v-if="item.image.ipfs"
                    >   
                    <template v-slot:loading><u-loading ></u-loading></template>
                    <template v-slot:error><view style="font-size: 28rpx;">加载失败</view></template>
                    </u-image>
                </view>
                <u-image
                    width="300rpx"
                    height="300rpx"
                    :src="postInfo.image"
                    v-if="postInfo.image"
                ></u-image>
                <u-link 
                    v-if="postInfo.url"
                    :href="postInfo.url" 
                    color="#f04a82"
                >
                {{ postInfo.simpleUrl }}
                </u-link>

                <view class="reward" v-if="postInfo.rewardList.length > 0">
                    <view
                        class="reward-list"
                        v-for="(item, index) in postInfo.rewardList.slice(0, 6)"
                        :key="index"
                    >
                        <Icon
                            icon="ri:thumb-up-fill"
                            color="#f04a82"
                            class="mr-6"
                        />
                        {{ item.nickname }} [ {{ item.sender_id.slice(-4) }} ]
                        {{ $t('components.reward') }}
                        <text class="name">
                            {{ balanceFormat(item.amount, 1) }}
                        </text>
                        WTT
                    </view>
                    <u-gap
                        height="5"
                        v-if="postInfo.rewardList.length > 6"
                    ></u-gap>
                    <view
                        class="more"
                        v-if="postInfo.rewardList.length > 6"
                        @click="rewardRecordShow = true"
                    >
                        {{ $t('components.viewMore') }}
                    </view>
                </view>
                <view class="more">
                    <Icon
                        :icon="postInfo.isStar ? 'ph:star-fill' : 'ph:star'"
                        width="18"
                        class="mr-24"
                        :color="postInfo.isStar ? '#ffc107' : '#666'"
                        @tap="star"
                    />
                    <Icon
                        icon="ant-design:copy-outlined"
                        width="18"
                        class="mr-24"
                        color="#666"
                        id="copy"
                        @click="copy"
                    />
                    <Icon
                        icon="ri:share-line"
                        width="18"
                        class="mr-24"
                        color="#666"
                        id="share"
                        @click="share"
                    />
                    <Icon
                            icon="fe:bar-chart"
                            width="18"
                            class="mr-10 "
                            color="#666"
                        />
                    <text class="read">
                        {{ postInfo.read }}
                    </text>
                </view>
            </view>
        </view>
        <RewardRecord
            v-model="rewardRecordShow"
            :record="postInfo.rewardList"
        ></RewardRecord>
    </view>
</template>

<style lang="scss" scoped>
.forum {
    &-item {
        background-color: #fff;
        margin-bottom: 20rpx;
        border-bottom: 2rpx solid #e9e9e9;
        position: relative;
        .focus {
            position: absolute;
            right: 0;
            top: 0;
            width: 0;
            height: 0;
            border-top: 40rpx solid #f04a82;
            border-left: 40rpx solid transparent;
            .star {
                position: absolute;
                transform: scale(0.65);
                right: 0rpx;
                top: -38rpx;
            }
        }
        .user-area {
            padding: 20rpx;
            display: flex;

            .head-box {
                margin-right: 20rpx;
                position: relative;
            }

            .user-info {
                display: flex;
                align-items: center;
                flex-wrap: wrap;
                width: 100%;

                .user {
                    width: 100%;
                    display: flex;
                    justify-content: space-between;
                    align-items: center;
                }

                .time {
                    font-size: 24rpx;
                    color: #91908e;
                    width: 100%;
                    text {
                        color: #999;
                        margin-right: 20rpx;
                    }
                }
            }
        }

        .main-content {
            margin: 0 30rpx 0rpx 30rpx;
            padding: 0 0 25rpx 0;
            font-size: 26rpx;

            .text-content {
                font-size: 32rpx;
                margin-bottom: 20rpx;
                word-wrap: break-word;
                word-break: normal;
                overflow: hidden;
                ::v-deep .topic-text {
                    color: #f04a82;
                }
                ::v-deep .mentions-text {
                    color: #f04a82;
                }
            }
            .reward {
                background: #f1f1f1;
                padding: 14rpx 30rpx;
                .reward-list {
                    margin: 10rpx 0;
                    .name {
                        color: #f04a82;
                    }
                }
            }
            .more {
                padding: 20rpx 0 0 0;
                border-top: 2rpx solid #e5e5e5;
                color: #666;
                text-align: right;
                .read {
                    vertical-align: top;
                }
            }
        }
    }
}
</style>
