<template>
    <n-gi :span="24">
        <n-upload
            :disabled="loading === true"
            multiple
            :show-file-list="true"
            list-type="image-card"
            v-model:file-list="fileList"
            :custom-request="customRequest"
            @before-upload="beforeUpload"
            directory-dnd
            accept="image/*"
        >
        </n-upload>
    </n-gi>
    <n-gi :span="24" style="margin-top: 10px">
        <n-spin description="图片预处理" :show="imageReaderLoading">
            <n-space align="center" justify="center">
                <n-button
                    :loading="loading"
                    type="error"
                    :disabled="fileList.length === 0"
                    @click="startOcr"
                >
                    识别
                </n-button>

                <n-button
                    tertiary
                    type="default"
                    :disabled="fileList.length === 0 || loading === true"
                    @click="clear"
                >
                    清空
                </n-button>
                <div v-if="loading === true">
                    线程数：{{ Object.keys(log).length }}
                </div>
            </n-space>
        </n-spin>
    </n-gi>
    <n-divider />
    <n-gi :span="24" style="height: calc(100vh - 400px)">
        <n-tabs size="large" type="line" animated justify-content="start">
            <template #suffix>
                <n-space justify="end">
                    <n-button
                        type="primary"
                        :disabled="tableData.length === 0"
                        @click="downloadCsv"
                        >下载csv
                    </n-button>
                </n-space>
            </template>
            <n-tab-pane name="before" tab="解析后">
                <n-data-table
                    v-if="loading === true"
                    size="small"
                    style="height: 100%"
                    max-height="height: 100%"
                    :scroll-x="200"
                    :columns="logColumns"
                    :data="logData"
                    :pagination="false"
                    :bordered="true"
                    striped
                />
                <n-data-table
                    v-else
                    size="small"
                    style="height: 100%"
                    max-height="height: 100%"
                    :loading="loading"
                    :columns="columns"
                    :data="tableData"
                    :pagination="false"
                    :bordered="true"
                    striped
                />
            </n-tab-pane>
            <n-tab-pane name="json" tab="JSON">
                <n-input
                    style="text-align: left"
                    :rows="19"
                    :loading="loading"
                    readonly
                    type="textarea"
                    v-model:value="tableDataStr"
                    placeholder=""
                ></n-input>
            </n-tab-pane>
            <n-tab-pane name="origin" tab="文本">
                <n-input
                    style="text-align: left"
                    :rows="19"
                    :loading="loading"
                    readonly
                    type="textarea"
                    v-model:value="text"
                    placeholder=""
                ></n-input>
            </n-tab-pane>
            <n-tab-pane name="option" tab="设置">
                <n-grid :x-gap="12" :y-gap="10">
                    <n-gi span="12">
                        <n-input-number
                            :disabled="loading"
                            :step="1"
                            :precision="0"
                            style="width: 100%"
                            size="large"
                            v-model:value="config.width"
                            :min="0"
                        >
                            <template #prefix>截图宽度</template>
                        </n-input-number>
                    </n-gi>
                    <n-gi :span="12">
                        <n-input-number
                            :disabled="loading"
                            :step="1"
                            :precision="0"
                            style="width: 100%"
                            size="large"
                            v-model:value="config.height"
                            :min="0"
                        >
                            <template #prefix>截图高度</template>
                        </n-input-number>
                    </n-gi>
                    <n-gi :span="12">
                        <n-input-number
                            :disabled="loading"
                            :step="1"
                            :precision="0"
                            style="width: 100%"
                            size="large"
                            v-model:value="config.roi.max"
                            :min="0"
                            :max="255"
                        >
                            <template #prefix>二值化阙值</template>
                        </n-input-number>
                    </n-gi>
                    <n-gi :span="12">
                        <n-input-number
                            :disabled="loading"
                            :step="1"
                            :precision="0"
                            style="width: 100%"
                            size="large"
                            v-model:value="config.roi.replace"
                            :min="0"
                            :max="255"
                        >
                            <template #prefix>替换颜色</template>
                        </n-input-number>
                    </n-gi>
                    <n-gi :span="12">
                        <n-input-number
                            :disabled="loading"
                            :step="1"
                            :precision="0"
                            style="width: 100%"
                            size="large"
                            v-model:value="config.crop.x1"
                            :min="0"
                            :max="config.width"
                        >
                            <template #prefix>识别区坐标x</template>
                        </n-input-number>
                    </n-gi>
                    <n-gi :span="12">
                        <n-input-number
                            :disabled="loading"
                            :step="1"
                            :precision="0"
                            style="width: 100%"
                            size="large"
                            v-model:value="config.crop.y1"
                            :min="0"
                            :max="config.height"
                        >
                            <template #prefix>识别区坐标y</template>
                        </n-input-number>
                    </n-gi>
                    <n-gi :span="12">
                        <n-input-number
                            :disabled="loading"
                            :step="1"
                            :precision="0"
                            style="width: 100%"
                            size="large"
                            v-model:value="config.crop.x2"
                            :min="0"
                            :max="config.width - config.crop.x1"
                        >
                            <template #prefix>识别区宽度</template>
                        </n-input-number>
                    </n-gi>
                    <n-gi :span="12">
                        <n-input-number
                            :disabled="loading"
                            :step="1"
                            :precision="0"
                            style="width: 100%"
                            size="large"
                            v-model:value="config.crop.y2"
                            :min="0"
                            :max="config.height - config.crop.y1"
                        >
                            <template #prefix>识别区高度</template>
                        </n-input-number>
                    </n-gi>
                    <n-divider />
                    <n-gi span="24" style="margin-top: 10px">
                        <div style="margin-bottom: 10px">预设：</div>
                        <div>
                            <n-space>
                                <n-button
                                    type="primary"
                                    @click="loadConfig('iPhone11')"
                                >
                                    iPhone11
                                </n-button>
                            </n-space>
                        </div>
                    </n-gi>
                </n-grid>
            </n-tab-pane>
        </n-tabs>
    </n-gi>
</template>

<script>
    import { reactive } from 'vue';
    import Tesseract from 'tesseract.js';
    import { Archive } from '@vicons/ionicons5';
    import { useMessage } from 'naive-ui';

    export default {
        name: 'Ocr',
        setup() {
            const config = {
                width: 828,
                height: 1792,
                crop: {
                    x1: 0,
                    y1: 185,
                    x2: 384,
                    y2: 1388,
                },
                roi: {
                    max: 180,
                    replace: 255,
                },
            };
            let message = useMessage();
            return {
                message,
                config: reactive(config),
            };
        },
        components: {
            Archive,
        },
        data() {
            return {
                // 预设配置
                defaultConfig: {
                    iPhone11: {
                        width: 828,
                        height: 1792,
                        crop: {
                            x1: 0,
                            y1: 185,
                            x2: 384,
                            y2: 1388,
                        },
                        roi: {
                            max: 180,
                            replace: 255,
                        },
                    },
                },
                fileList: [],
                loading: false,
                imageReaderLoading: false,
                columns: [
                    {
                        title: '#',
                        key: 'index',
                        width: 60,
                        render: (_, index) => {
                            return `${index + 1}`;
                        },
                    },
                    {
                        title: '联系人',
                        key: 'concat',
                    },
                    {
                        title: '归属地',
                        key: 'locale',
                    },
                    {
                        title: '关联图片',
                        key: 'image',
                    },
                ],
                tableData: [],
                log: {},
                text: '',
                logColumns: [
                    {
                        title: '#',
                        key: 'index',
                        width: 60,
                        render: (_, index) => {
                            return `${index + 1}`;
                        },
                    },
                    {
                        title: '进程ID',
                        key: 'userJobId',
                    },
                    {
                        title: '进度',
                        key: 'progress',
                        render: (value) => {
                            return `${Math.round(
                                Number(value.progress) * 100
                            )}%`;
                        },
                    },
                    {
                        title: '状态',
                        key: 'status',
                    },
                ],
                loadImgTimmer: null,
            };
        },
        methods: {
            readImg(file) {
                return new Promise((resolve) => {
                    const reader = new FileReader();
                    reader.onload = (e) => {
                        const $image = new Image();
                        $image.onload = (res) => {
                            setTimeout(() => {
                                try {
                                    const imgMat = cv.imread($image);
                                    // 将图片缩放至指定尺寸
                                    const resizedMat = new cv.Mat();
                                    const size = new cv.Size(
                                        this.config.width,
                                        this.config.height
                                    );
                                    cv.resize(
                                        imgMat,
                                        resizedMat,
                                        size,
                                        0,
                                        0,
                                        cv.INTER_AREA
                                    );
                                    // 裁剪指定区域
                                    const rect = new cv.Rect(
                                        this.config.crop.x1,
                                        this.config.crop.y1,
                                        this.config.crop.x2,
                                        this.config.crop.y2
                                    );
                                    const roiMat = resizedMat.roi(rect);
                                    // 二值化
                                    cv.cvtColor(
                                        roiMat,
                                        roiMat,
                                        cv.COLOR_RGBA2GRAY
                                    );
                                    cv.threshold(
                                        roiMat,
                                        roiMat,
                                        this.config.roi.max,
                                        this.config.roi.replace,
                                        cv.THRESH_BINARY
                                    );

                                    const canvas =
                                        document.createElement('canvas');
                                    const ctx = canvas.getContext('2d');
                                    // 在画布上显示图像
                                    cv.imshow(canvas, roiMat);
                                    // 释放Mat对象
                                    roiMat.delete();
                                    resizedMat.delete();
                                    imgMat.delete();
                                    // 获取base64编码
                                    const base64 = canvas.toDataURL();
                                    resolve(base64);
                                } catch (e) {
                                    console.log(e);
                                    this.message.warning(
                                        `Opencv error: ${e.toString()}, 请检查参数`
                                    );
                                    resolve(false);
                                }
                            }, 50);
                        };
                        $image.src = e.target.result;
                    };
                    reader.readAsDataURL(file);
                });
            },
            customRequest() {
                if (this.loadImgTimmer) {
                    clearTimeout(this.loadImgTimmer);
                }
                this.loadImgTimmer = setTimeout(() => {
                    Promise.all(
                        this.fileList.map((item) => {
                            return new Promise(async (resolve) => {
                                if (
                                    item.status !== 'finished' &&
                                    item.status !== 'uploading'
                                ) {
                                    this.imageReaderLoading = true;
                                    item.status = 'uploading';
                                    item.percentage = 0;
                                    const base64 = await this.readImg(
                                        item.file
                                    );
                                    if (base64) {
                                        item.url = base64;
                                        item.status = 'finished';
                                        item.percentage = 100;
                                        resolve();
                                    } else {
                                        item.status = 'error';
                                        item.percentage = 0;
                                        resolve();
                                    }
                                } else {
                                    resolve();
                                }
                            });
                        })
                    ).then(() => {
                        this.imageReaderLoading = false;
                    });
                }, 50);
            },
            startOcr() {
                this.loading = true;
                this.tableData = [];
                this.log = {};
                this.text = '';
                let tableData = [];
                Promise.all(
                    this.fileList.map((item) => {
                        return new Promise(async (resolve, reject) => {
                            const base64 = item.url;
                            const fileName = item.name;
                            if (base64) {
                                const recognizeImage = async (imageUrl) => {
                                    const worker = await Tesseract.createWorker(
                                        {
                                            workerPath: 'js/worker.min.js',
                                            corePath:
                                                'js/tesseract-core.wasm.js',
                                            langPath: 'langs/',
                                            logger: (m) => {
                                                this.log[m.userJobId] = m;
                                            },
                                        }
                                    );
                                    // await worker.load();
                                    await worker.loadLanguage('chi_sim');
                                    await worker.initialize('chi_sim');
                                    const {
                                        data: { text },
                                    } = await worker.recognize(imageUrl);
                                    await worker.terminate();

                                    if (!text) {
                                        this.message.warning('未识别到文字');
                                        resolve();
                                        return;
                                    }
                                    this.text += text;
                                    let arr = text.split('\n');
                                    arr = arr.filter((item) => {
                                        return !!item;
                                    });
                                    arr = arr.map((item) =>
                                        item.replace(/\s/g, '')
                                    );
                                    const data = [];
                                    let obj = {};
                                    arr.forEach((arrItem, index) => {
                                        if (index % 2 === 0) {
                                            obj.concat = arrItem;
                                        } else {
                                            obj.locale = arrItem;
                                            obj.image = fileName;
                                            data.push(obj);
                                            obj = {};

                                            if (index === arr.length - 1) {
                                                tableData =
                                                    tableData.concat(data);

                                                resolve(data);
                                            }
                                        }
                                    });
                                };
                                await recognizeImage(base64);
                            } else {
                                resolve();
                            }
                        });
                    })
                ).then(() => {
                    this.loading = false;
                    this.log = {};
                    this.tableData = tableData;
                });
            },
            async beforeUpload(e) {
                const finder = this.fileList.find(
                    (item) => item.fullPath === e.file.fullPath
                );
                if (finder) {
                    return false;
                }

                return true;
            },
            clear() {
                this.fileList = [];
                this.tableData = [];
                this.log = {};
                this.text = '';
            },
            downloadCsv() {
                const tableData = [...this.tableData];
                let content = '\uFEFF'; // bom头

                const th = this.columns.map((item) => item.title);
                content += th.join(',') + '\r\n';

                const keys = this.columns.map((item) => item.key);
                tableData.forEach((item, index) => {
                    item.index = index + 1;
                    const arr = keys.map((thItem) => `"${item[thItem]}"`);
                    content += arr.join(',') + '\r\n';
                });
                const blob = new Blob([content], {
                    type: 'text/csv;charset=utf-8',
                });
                const link = document.createElement('a');
                link.download = 'miss-call.csv';
                link.href = window.URL.createObjectURL(blob);
                link.click();
            },
            loadConfig(type) {
                if (this.defaultConfig[type]) {
                    this.config.width = this.defaultConfig[type].width;
                    this.config.height = this.defaultConfig[type].height;
                    this.config.crop.x1 = this.defaultConfig[type].crop.x1;
                    this.config.crop.y1 = this.defaultConfig[type].crop.y1;
                    this.config.crop.x2 = this.defaultConfig[type].crop.x2;
                    this.config.crop.y2 = this.defaultConfig[type].crop.y2;
                    this.config.roi.max = this.defaultConfig[type].roi.max;
                    this.config.roi.replace =
                        this.defaultConfig[type].roi.replace;

                    this.message.success(`参数已设置为${type}`);
                } else {
                    this.message.warning('配置不存在');
                }
            },
        },
        computed: {
            logData() {
                return Object.keys(this.log)
                    .filter((item) => this.log[item].progress !== 1)
                    .map((item) => this.log[item]);
            },
            tableDataStr() {
                return JSON.stringify(this.tableData);
            },
        },
    };
</script>

<style scoped></style>
