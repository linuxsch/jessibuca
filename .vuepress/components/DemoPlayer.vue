<template>
    <div class="root">
        <div class="container-shell">
            <div id="container" class="container" ref="container"></div>
            <div class="input">
                <div>输入URL：</div>
                <input autocomplete="on" ref="playUrl"
                       value=""/>
                <button v-if="!playing" @click="play">播放</button>
                <button v-else @click="pause">停止</button>
            </div>
            <div class="input" v-if="loaded">
                <button @click="destroy">销毁</button>
                <button @click="fullscreen">全屏</button>
                <button @click="screenShot">截图</button>
                <div style="line-height: 30px">
                    <input type="checkbox" ref="operateBtns" v-model="showOperateBtns" @change="restartPlay"><span>操作按钮</span>
                    <input type="checkbox" ref="operateBtns" v-model="showBandwidth" @change="restartPlay"><span>网速</span>
                    <span v-if="performance">性能：{{performance}}</span>
                </div>

            </div>
            <div class="option">
                <span>缓冲(秒):</span>
                <input style="width:50px" type="number" ref="buffer" value="0.2" @change="changeBuffer">
                <input type="checkbox" ref="wasm" @change="changeWasm"><span>wasm</span>
                <select ref="vc" @change="changeVC">
                    <option selected>h264</option>
                    <option>h265</option>
                </select>
                <input type="checkbox" ref="resize" @change="changeResize"><span>自适应</span>
            </div>
        </div>
    </div>
</template>
<script>
    import Jessibuca from "./renderer";

    export default {
        name: "DemoPlayer",
        props: {},
        data() {
            return {
                jessibuca: null,
                wasm: false,
                vc: "ff",
                playing: false,
                loaded: false, // mute
                showOperateBtns:false,
                showBandwidth:false,
                err: "",
                speed: 0,
                muting: true,
                performance:''
            };
        },
        computed: {
            decoder() {
                return this.vc + (this.wasm ? "_wasm" : "") + ".js"
            },
            speedShow() {

            }
        },
        watch: {
            decoder(v) {
                this.restartPlay();
            }
        },
        mounted() {
            this.create()
            window.onerror = msg => (this.err = msg);
        },
        destroyed() {
            this.jessibuca.destroy();
        },
        methods: {
            create(options) {
                options = options || {};
                this.jessibuca = new Jessibuca(Object.assign({
                    container: this.$refs.container,
                    decoder: this.decoder,
                    videoBuffer: Number(this.$refs.buffer.value),
                    isResize: false,
                    text: '',
                    background: 'https://seopic.699pic.com/photo/40011/0709.jpg_wh1200.jpg',
                    loadingText: '加载中',
                    debug: true,
                    showBandwidth: this.showBandwidth, // 显示网速
                    operateBtns: {
                        fullscreen: this.showOperateBtns,
                        screenshot: this.showOperateBtns,
                        play: this.showOperateBtns,
                        audio: this.showOperateBtns
                    }
                }, options));
                this.jessibuca.onLog = msg => console.log('onLog', msg);
                this.jessibuca.onLoad = msg => console.log('onLoad');
                this.jessibuca.onRecord = msg => console.log('onRecord', msg);
                this.jessibuca.onPause = () => console.log('onPause');
                this.jessibuca.onPlay = () => console.log('onPlay');
                this.jessibuca.onFullscreen = msg => console.log('onFullscreen', msg);
                this.jessibuca.onMute = msg => console.log('onMute', msg);
                this.jessibuca.onInitSize = ()=>console.log('onInitSize');
                // this.jessibuca.onTimeUpdate = (ts)=> console.log('onTimeUpdate',ts);
                var _this = this;
                this.jessibuca.on('load', function () {
                    console.log('on load');
                });

                this.jessibuca.on('log', function (msg) {
                    console.log('on log', msg);
                });
                this.jessibuca.on('record', function (msg) {
                    console.log('on record:', msg);
                });
                this.jessibuca.on('pause', function () {
                    console.log('on pause');
                    _this.playing = false;
                });
                this.jessibuca.on('play', function () {
                    console.log('on play');
                    _this.playing = true;
                });
                this.jessibuca.on('fullscreen', function (msg) {
                    console.log('on fullscreen', msg);
                });

                this.jessibuca.on('mute', function (msg) {
                    console.log('on mute', msg);
                });

                this.jessibuca.on('mute', function (msg) {
                    console.log('on mute2', msg);
                });

                this.jessibuca.on('audioInfo', function (msg) {
                    console.log('audioInfo', msg);
                });


                this.jessibuca.on('bps', function (bps) {
                    // console.log('bps', bps);
                });

                this.jessibuca.on('timeUpdate',function (ts) {
                    // console.log('timeUpdate',ts);
                })

                this.jessibuca.on('videoInfo', function (info) {
                    console.log('videoInfo', info);
                })

                this.jessibuca.on('error', function (error) {
                    console.log('error', error)
                });

                this.jessibuca.on('timeout', function () {
                    console.log('timeout');
                });

                this.jessibuca.on('stats',function (stats) {
                    console.log('stats',JSON.stringify(stats));
                })

                this.jessibuca.on('performance',function (performance) {
                    var show = '卡顿';
                    if(performance === 2){
                        show = '非常流畅'
                    }
                    else if(performance === 1){
                        show = '流畅'
                    }
                    console.log('stats',show);
                    _this.performance = show;
                })

                console.log(this.jessibuca);
            },
            play() {
                // this.jessibuca.onPlay = () => (this.playing = true);
                this.jessibuca.on('play', () => {
                    this.playing = true;
                    this.loaded = true;
                });


                if (this.$refs.playUrl.value) {
                    if (this.jessibuca.hasLoaded()) {
                        this.jessibuca.play(this.$refs.playUrl.value);
                    } else {
                        this.jessibuca.on('load', () => {
                            this.jessibuca.play(this.$refs.playUrl.value);
                        })
                    }
                }
            },
            mute() {
                this.jessibuca.mute();
                this.muting = true;
            },
            cancelMute() {
                this.jessibuca.cancelMute();
                this.muting = false;
            },

            pause() {
                this.jessibuca.pause();
                this.playing = false;
                this.err = "";
                this.performance = '';
            },

            destroy() {
                if (this.jessibuca) {
                    this.jessibuca.destroy();
                }
                this.create();
                this.playing = false;
                this.loaded = false;
                this.performance = '';
            },

            fullscreen() {
                this.jessibuca.setFullscreen(true);
            },

            screenShot() {
                this.jessibuca.screenshot();
            },

            changeVC() {
                this.vc = ["ff", "libhevc_aac"][this.$refs.vc.selectedIndex]
            },

            changeWasm() {
                this.wasm = this.$refs.wasm.checked
            },

            restartPlay() {
                this.destroy();
                this.play();
            },

            changeBuffer() {
                this.jessibuca.setBufferTime(Number(this.$refs.buffer.value));
            },

            changeResize() {
                const value = this.$refs.resize.checked ? 1 : 0;
                this.jessibuca.setScaleMode(value);
            }
        }
    };
</script>
<style>
    .root {
        display: flex;
        place-content: center;
    }

    .container-shell {
        background: gray;
        padding: 30px 4px 10px 4px;
        border: 2px solid black;
        width: auto;
        position: relative;
        border-radius: 5px;
        box-shadow: 0 10px 20px;
    }

    .container-shell:before {
        content: "jessibuca demo player";
        position: absolute;
        color: darkgray;
        top: 4px;
        left: 10px;
        text-shadow: 1px 1px black;
    }

    .container {
        background: rgb(13, 14, 27);
        width: 640px;
        height: 398px;
    }

    .input {
        display: flex;
        margin-top: 10px;
        color: white;
        place-content: stretch;
    }

    .input2 {
        bottom: 0px;
    }


    .input input {
        flex: auto;
    }

    .err {
        position: absolute;
        top: 40px;
        left: 10px;
        color: red;
    }

    .option {
        position: absolute;
        top: 4px;
        right: 10px;
        display: flex;
        place-content: center;
    }

    .option span {
        color: white;
    }

    @media (max-width: 720px) {
        .container {
            width: 90vw;
            height: 52.7vw;
        }
    }
</style>
