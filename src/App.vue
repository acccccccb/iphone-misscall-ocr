<script setup>
    // This starter template is using Vue 3 <script setup> SFCs
    // Check out https://vuejs.org/api/sfc-script-setup.html#script-setup
    import Ocr from './components/Ocr.vue';
    import { defineComponent, ref } from 'vue';
    import { darkTheme, lightTheme } from 'naive-ui';
</script>

<template>
    <n-config-provider :theme="darkTheme">
        <n-layout content-style="min-height: 100vh">
            <n-layout-header class="header">
                iphone-misscall-ocr
            </n-layout-header>
            <n-layout-content
                class="content"
                content-style="width: 100%;height: calc(100vh - 130px);padding: 10px;"
                :native-scrollbar="false"
            >
                <n-grid x-gap="10" :y-gap="10" :cols="24">
                    <n-gi :span="24">
                        <n-message-provider>
                            <Ocr />
                        </n-message-provider>
                    </n-gi>
                </n-grid>
            </n-layout-content>
            <n-layout-footer class="footer">
                <a
                    target="_blank"
                    href="https://gitee.com/GLUESTICK/iphone-misscall-ocr"
                >
                    Gitee
                </a>
                |
                <a
                    target="_blank"
                    href="https://github.com/acccccccb/iphone-misscall-ocr"
                >
                    Github
                </a>
            </n-layout-footer>
        </n-layout>
    </n-config-provider>
</template>

<style>
    #app {
        font-family: Avenir, Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        color: #2c3e50;
    }
    html,
    body,
    #app {
        padding: 0;
        margin: 0;
        height: 100%;
    }
    .header {
        line-height: 80px;
        height: 80px;
        text-align: center;
        font-size: 1.6em;
    }
    .content {
        min-height: calc(100vh - 130px);
        display: flex;
        justify-content: center;
        /*align-items: center;*/
    }
    .footer {
        line-height: 50px;
        height: 50px;
        text-align: center;
    }
    .footer a {
        color: #fff;
    }
</style>
