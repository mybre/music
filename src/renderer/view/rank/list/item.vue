<template>
    <li
        :class="{ [s.item]: true, [s.onlyCover]: info && !info.list.length }"
        @click="go2RankList"
    >
        <template v-if="info">
            <div :class="s.cover">
                <div :class="s.play">
                    <play-loading v-if="loading"></play-loading>
                    <Icon
                        type="play1"
                        @click.stop="playList"
                        :class="s.icon"
                        v-else
                    ></Icon>
                </div>
            </div>
            <div :class="s.img">
                <img :src="info.cover | image(vendor, -1)" />
            </div>
            <ul :class="s.songs" v-if="list.length">
                <li v-for="(song, index) in list" :class="s.song">
                    <span>{{ index + 1 }}</span>
                    <span>{{ song.name }}</span
                    >-&nbsp;
                    <span>
                        <template v-for="singer in song.artists">
                            {{ singer.name }}
                        </template>
                    </span>
                </li>
            </ul>
        </template>
    </li>
</template>
<script>
import playLoading from './play-loading.vue'
import eventBus from '../eventBus'

export default {
    props: {
        info: {
            default: null,
        },
        vendor: String,
    },
    components: {
        playLoading,
    },
    data() {
        return {
            loading: false,
        }
    },
    computed: {
        list() {
            return this.info ? this.info.list.slice(0, 3) : []
        },
    },
    methods: {
        // 播放排行榜
        async playList() {
            this.loading = true
            try {
                const { status, data } = await this.$musicApi[
                    this.vendor
                ].getTopList(this.info.id)
                if (status) {
                    data.list.forEach(item => {
                        item.songId = item.id
                        item.vendor = this.vendor
                    })
                    this.$store.dispatch('play/play', {
                        info: data.list[0],
                        playlist: data.list,
                    })
                } else {
                    this.$message.warning('无法获取榜单详情')
                }
            } catch (e) {
                console.warn(e)
            }
            this.loading = false
        },
        // 跳转至排行榜详情
        go2RankList() {
            this.$router.push({
                name: 'rank.detail',
                params: {
                    id: this.info.id,
                },
                query: {
                    vendor: this.vendor,
                },
            })
        },
    },
}
</script>
<style lang="scss" module="s">
.item {
    display: inline-flex;
    width: calc(50% - 10px);
    height: 120px;
    margin-bottom: 20px;
    position: relative;
    margin-right: 10px;
    background-color: #f9f9f9;

    $coverWidth: 120px;

    &.onlyCover {
        width: $coverWidth;
    }
    .cover {
        display: flex;
        justify-content: center;
        align-items: center;
        position: absolute;
        left: 0;
        bottom: 0;
        color: white;
        z-index: 2;
        transition: all 0.2s;
        text-align: center;
        user-select: none;
        cursor: pointer;
        .play {
            display: none;
            position: absolute;
            left: 0;
            top: 0;
            right: 0;
            bottom: 0;
            margin: auto;
            align-items: center;
            justify-content: center;
            .icon {
                color: white;
                font-size: 40px;
                padding: 8px;
                padding-left: 12px;
                border-radius: 50%;
                background-color: #29c777;
                transition: all 0.2s;
                &:hover {
                    transition: all 0.2s;
                    background-color: #26a866;
                }
            }
        }
        &:hover {
            transition: all 0.2s;
            background-color: rgba(0, 0, 0, 0.2);
            .play {
                display: flex;
            }
            & + .img > img {
                transform: scale(1.2, 1.2);
            }
        }
    }
    .img,
    .cover {
        width: $coverWidth;
        height: 100%;
    }
    .img {
        overflow: hidden;
        img {
            transition: all 0.2s;
            width: 100%;
            height: 100%;
        }
    }
    .songs {
        display: flex;
        flex-direction: column;
        justify-content: space-around;
        padding: 20px 16px;
        list-style: none;
        background-color: #fbfbfb;
        cursor: pointer;
        width: calc(100% - #{$coverWidth});
        &:hover {
            background-color: #f2f2f2;
        }
        .song {
            font-size: 13px;
            color: #231919;
            display: flex;
            align-items: center;
            & > span {
                white-space: nowrap;
                overflow: hidden;
                text-overflow: ellipsis;
                vertical-align: middle;
                line-height: 1;
            }
            span:nth-child(1) {
                margin-right: 4px;
            }
            span:nth-child(2) {
                width: 120px;
            }
            span:nth-child(3) {
                width: 75px;
            }
        }
    }
}
</style>
