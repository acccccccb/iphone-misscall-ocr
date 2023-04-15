<template>
    <div>
        <n-grid style="margin-top: 30px" x-gap="24">
            <n-gi :span="8">
                <n-scrollbar style="height: calc(100vh - 400px)">
                    <n-upload
                        :disabled="loading === true"
                        multiple
                        :show-file-list="true"
                        list-type="image-card"
                        v-model:file-list="fileList"
                        @update:file-list="handleFileListChange"
                        directory-dnd
                        accept="image/*"
                        :custom-request="customRequest"
                    >
                    </n-upload>
                </n-scrollbar>
            </n-gi>

            <n-gi :span="16">
                <n-data-table
                    v-if="loading === true"
                    size="small"
                    style="height: calc(100vh - 400px)"
                    max-height="calc(100vh - 400px)"
                    :columns="logColumns"
                    :data="logData"
                    :pagination="false"
                    :bordered="true"
                    striped
                />
                <n-data-table
                    v-else
                    size="small"
                    style="height: calc(100vh - 400px)"
                    max-height="calc(100vh - 400px)"
                    :loading="loading"
                    :columns="columns"
                    :data="tableData"
                    :pagination="false"
                    :bordered="true"
                    striped
                />
            </n-gi>
            <n-gi :span="24" style="margin-top: 30px">
                <n-space align="center">
                    <n-button
                        :loading="loading"
                        size="large"
                        type="error"
                        :disabled="fileList.length === 0"
                        @click="startOcr"
                        >识别
                    </n-button>
                    <n-button
                        size="large"
                        type="primary"
                        secondary
                        :disabled="tableData.length === 0"
                        @click="downloadCsv"
                        >下载csv
                    </n-button>
                    <n-button
                        tertiary
                        size="large"
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
            </n-gi>
            <n-gi :span="24" style="margin-top: 30px; margin-bottom: 30px">
                <n-space align="center">
                    <n-input-number
                        :step="1"
                        :precision="0"
                        style="width: 200px"
                        size="large"
                        v-model:value="config.width"
                        :min="0"
                    >
                        <template #prefix>截图宽度</template>
                    </n-input-number>
                    <n-input-number
                        :step="1"
                        :precision="0"
                        style="width: 200px"
                        size="large"
                        v-model:value="config.height"
                        :min="0"
                    >
                        <template #prefix>截图高度</template>
                    </n-input-number>

                    <n-input-number
                        :step="1"
                        :precision="0"
                        style="width: 200px"
                        size="large"
                        v-model:value="config.roi.max"
                        :min="0"
                        :max="255"
                    >
                        <template #prefix>二值化阙值</template>
                    </n-input-number>
                    <n-input-number
                        :step="1"
                        :precision="0"
                        style="width: 200px"
                        size="large"
                        v-model:value="config.roi.replace"
                        :min="0"
                        :max="255"
                    >
                        <template #prefix>替换颜色</template>
                    </n-input-number>
                </n-space>
                <n-space style="margin-top: 10px">
                    <n-input-number
                        :step="1"
                        :precision="0"
                        style="width: 200px"
                        size="large"
                        v-model:value="config.crop.x1"
                        :min="0"
                        :max="config.width"
                    >
                        <template #prefix>识别区域x1</template>
                    </n-input-number>
                    <n-input-number
                        :step="1"
                        :precision="0"
                        style="width: 200px"
                        size="large"
                        v-model:value="config.crop.y1"
                        :min="0"
                        :max="config.height"
                    >
                        <template #prefix>识别区域y1</template>
                    </n-input-number>
                    <n-input-number
                        :step="1"
                        :precision="0"
                        style="width: 200px"
                        size="large"
                        v-model:value="config.crop.x2"
                        :min="config.crop.x1"
                        :max="config.width"
                    >
                        <template #prefix>识别区域x2</template>
                    </n-input-number>
                    <n-input-number
                        :step="1"
                        :precision="0"
                        style="width: 200px"
                        size="large"
                        v-model:value="config.crop.y2"
                        :min="config.crop.y1"
                        :max="config.height"
                    >
                        <template #prefix>识别区域y2</template>
                    </n-input-number>
                </n-space>
            </n-gi>
        </n-grid>
    </div>
</template>

<script>
    import Tesseract from 'tesseract.js';
    import { Archive } from '@vicons/ionicons5';
    import { useMessage } from 'naive-ui';

    export default {
        name: 'Ocr',
        setup() {
            let message = useMessage();
            return {
                message,
            };
        },
        components: {
            Archive,
        },
        data() {
            return {
                config: {
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
                fileList: [],
                loading: false,
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
                                resolve($image);
                            }, 50);
                        };
                        $image.src = e.target.result;
                    };
                    reader.readAsDataURL(file);
                });
            },
            handleFileListChange(fileList) {
                this.fileList = fileList;
            },
            startOcr() {
                this.loading = true;
                this.tableData = [];
                this.log = {};
                let tableData = [];
                Promise.all(
                    this.fileList.map((item) => {
                        return new Promise(async (resolve) => {
                            const $image = await this.readImg(item.file);
                            $image.name = item.file.name;
                            if ($image) {
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
                                cv.cvtColor(roiMat, roiMat, cv.COLOR_RGBA2GRAY);
                                cv.threshold(
                                    roiMat,
                                    roiMat,
                                    this.config.roi.max,
                                    this.config.roi.replace,
                                    cv.THRESH_BINARY
                                );

                                const canvas = document.createElement('canvas');
                                const ctx = canvas.getContext('2d');
                                // 在画布上显示图像
                                cv.imshow(canvas, roiMat);

                                // 获取base64编码
                                const base64 = canvas.toDataURL();

                                const recognizeImage = async (imageUrl) => {
                                    const worker = await Tesseract.createWorker(
                                        {
                                            workerPath: '/js/worker.min.js',
                                            corePath:
                                                '/js/tesseract-core.wasm.js',
                                            langPath: '/js/langs/',
                                            logger: (m) => {
                                                this.log[m.userJobId] = m;
                                            },
                                        }
                                    );
                                    await worker.load();
                                    await worker.loadLanguage('chi_sim');
                                    await worker.initialize('chi_sim');
                                    const {
                                        data: { text },
                                    } = await worker.recognize(imageUrl);
                                    await worker.terminate();
                                    let arr = text.split('\n');
                                    arr = arr.filter((item) => {
                                        return !!item;
                                    });
                                    arr = arr.map((item) =>
                                        item.replace(/\s/g, '')
                                    );
                                    const data = [];
                                    let obj = {};
                                    arr.forEach((item, index) => {
                                        if (index % 2 === 0) {
                                            obj.concat = item;
                                        } else {
                                            obj.locale = item;
                                            obj.image = $image.name;
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

                                // 释放Mat对象
                                roiMat.delete();
                                resizedMat.delete();
                                imgMat.delete();
                            }
                        });
                    })
                ).then(() => {
                    this.loading = false;
                    // this.log = {};
                    this.tableData = tableData;
                });
            },
            async customRequest(e) {},
            clear() {
                this.fileList = [];
                this.tableData = [];
                this.log = {};
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
        },
        computed: {
            logData() {
                return Object.keys(this.log)
                    .filter((item) => this.log[item].progress !== 1)
                    .map((item) => this.log[item]);
            },
        },
    };
</script>

<style scoped></style>
