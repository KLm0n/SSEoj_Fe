<!-- eslint-disable vue/multi-word-component-names -->

<template>
    <div v-if="loadComplete">
        <div class="pCard">
            <div class="cardLeft">
                <div class="pId">
                    {{ id }}
                    <span 
                        v-if="problemDesc.pass_status" 
                        class="iconfont icon-duigou1"
                        style="margin-left: 6px; font-size: 18px;"
                    ></span>
                    
                </div>
                <div class="pName" :style="{ 'border-bottom': `3px solid ${getDifficultColor(problemDesc.difficulty)}` }">{{ problemDesc.name }}
                </div>
                <div class="source">题目来源：{{ problemDesc.source }}</div>
                <div class="difficulty">难度：<span :style="{ 'color': `${getDifficultColor(problemDesc.difficulty)}` }">Lv.{{
                    problemDesc.difficulty
                        }}</span></div>
            </div>

            <div class="cardRight">
                <div>时间限制：{{ transTime(problemDesc.time_limit) }}</div>
                <div>内存限制：{{ transMem(problemDesc.memory_limit) }}</div>
                <div class="star-block" @click="showStarForm">
                    <span>+ 收藏</span>
                    <span 
                        :class="problemDesc.star_status? 'iconfont icon-shoucang': 'iconfont icon-shoucangdanse'"
                        style="font-size: 22px; margin-left: 4px;"
                    >
                    </span>
                </div>
                <div class="study-plan-block" @click="addToStudyPlan">
                    + 加入做题计划
                </div>
            </div>
        </div>

        <div>
            <div class="node">题目描述</div>
            <div v-html="problemDesc.description"></div>
        </div>
        <el-divider />
        <div>
            <div class="node">输入格式</div>
            <div v-html="problemDesc.input_style"></div>
        </div>
        <el-divider />
        <div>
            <div class="node">输出格式</div>
            <div v-html="problemDesc.output_style"></div>
        </div>
        <el-divider />
        <div>
            <div class="node">数据范围</div>
            <div v-html="problemDesc.data_range"></div>
        </div>
        <el-divider />
        <div>
            <div class="node">输入输出样例</div>
            <div v-for="(item, index) in problemDesc.sample.inputs" :key="index" class="sampleBox">
                <div class="inputs">
                    <span>Input #{{ index }}</span>
                    <span class="iconfont icon-fuzhi" style="cursor: pointer;" @click="copyText(item)"></span>
                </div>
                <div class="sample" style="font-size: 14px;">{{ item }}</div>
                <div class="outputs">
                    <span>Output #{{ index }}</span>
                    <span class="iconfont icon-fuzhi" style="cursor: pointer;" @click="copyText(problemDesc.sample.outputs[index])"></span>
                </div>
                <div class="sample" style="font-size: 14px;">{{ problemDesc.sample.outputs[index] }}</div>
            </div>
        </div>
        <el-divider style="margin-bottom: 10px;" />
        <div>
            <el-collapse style="border: none;">
                <el-collapse-item style="--el-collapse-header-height:50px; vertical-align: center;">
                    <template #title>
                        <div class="node" style="">标签</div>
                    </template>
                    <el-tag v-for="(tag_index, index) in problemDesc.tags" :key="index" type="info" :disable-transitions="true"
                        color="#EAEAEA">
                        {{ tagsStore.idToTags[tag_index]?.name }}
                    </el-tag>

                </el-collapse-item>
            </el-collapse>
        </div>


        <el-dialog v-model="starFormVisible" style="width: 400px;">
            <div style="font-size: 20px; font-weight: 500; margin-bottom: 20px;">我的收藏</div>
            <el-scrollbar height="200px">
                <div style="margin-left: 20px;">
                    <el-checkbox label="默认题单" size="large" v-model="starDefault" >
                        <template #default>
                            <span class="iconfont icon-suoding1"></span>
                            <span style="margin-left: 3px; font-size:15px">默认题单</span>
                        </template>
                    </el-checkbox>
                </div>
                
                <el-checkbox-group v-model="starList">
                    <el-checkbox v-for="(item, index) in problemlists" 
                        :key="index" 
                        :value="item.id"
                        size="large"
                    >
                        <template #default>
                            <span v-if="!item.type" class="iconfont icon-suoding1"></span>
                            <span 
                                style="
                                    display: inline-block; 
                                    white-space: nowrap; 
                                    overflow: hidden; 
                                    text-overflow: ellipsis;
                                    width: 200px;
                                    margin-left: 3px;
                                    font-size: 15px;
                                "
                            >{{ item.title }}</span>
                        </template>
                    </el-checkbox>
                </el-checkbox-group>
            </el-scrollbar>
            <div style="display: flex; flex-direction: row;">
                <el-button @click="confirmStar" >确定</el-button>
            </div>
            
        </el-dialog>
    </div>

</template>

<script setup>
import { useRoute } from 'vue-router';
import { onMounted, ref } from 'vue';
import { transTime, transMem } from '@/utils/data_calculate';
import { ElMessage } from 'element-plus';
import { useTagsStore } from '@/stores/tagsStore';
import { getDifficultColor } from '@/utils/color';
import { getProblemDescAPI } from "@/apis/problem";
import { getCreateProblemListAPI } from '@/apis/user';
import { useUserStore } from '@/stores/userStore';
import { starToDefaultProblemAPI } from '@/apis/problem';
import { starToProblemListAPI, addToStudyPlanAPI } from '@/apis/problem';
import '@/assets/base-el-tag.css'

const route = useRoute()
const loadComplete = ref(false)
const id = route.params.id

const problemDesc = ref({})
const starFormVisible = ref(false)
const tagsStore = useTagsStore()
const userStore = useUserStore()
const problemlists = ref([])
const starDefault = ref(false)
const starList = ref([])

const getProblemDesc = async (id) => {
    const res = await getProblemDescAPI(id)
    problemDesc.value = res.data
    console.log(problemDesc)
}

// 获取自己创建题单
const getCreateProblemList = async() => {
    console.log(userStore?.userInfo?.id)
    if (!userStore?.userInfo?.id) {
        ElMessage.warning('请先登录')
    }
    const res = await getCreateProblemListAPI(userStore.userInfo.id)
    problemlists.value = res.data
}

// 确认收藏
const confirmStar = async() => {
    console.log(starList.value)
    if (starDefault.value) {
        await starToDefaultProblemAPI(id)
    }
    
    for (let item of starList.value) {
        await starToProblemListAPI({
            problem_id: id,
            problemlist_id: item
        });
    }
    if (starDefault.value || starList.value) {
        ElMessage.success('收藏成功')
        problemDesc.value.star_status = true
    }
    starDefault.value = false
    starFormVisible.value = false
}

// 加入做题计划
const addToStudyPlan = async() => {
    await addToStudyPlanAPI(id)
    ElMessage.success('加入成功')
}

onMounted(async () => {
    await getProblemDesc(id)
    await tagsStore.getTags()
    await getCreateProblemList()
    loadComplete.value = true
})

const copyText = async (txt) => {
    if ('clipboard' in navigator) {
        try {
            await navigator.clipboard.writeText(txt);
            ElMessage({
                message: '文本已复制到剪贴板',
                type: 'message',
            })
        } catch (err) {
            ElMessage({
                message: err,
                type: 'error',
            })
        }
    } else {
        ElMessage({
            message: '当前浏览器不支持 Clipboard API',
            type: 'error',
        })
    }
}

const showStarForm = () => {
    starFormVisible.value = true
}

</script>

<style scoped>
.pCard {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    margin-bottom: 10px
}

.cardLeft {
    display: flex;
    flex-direction: column;
    flex: 1;
    padding-right: 30px;
}

.cardRight {
    display: flex;
    flex-direction: column;
    color: #696666;
    font-size: 12px;
    padding-top: 10px;
    gap: 5px;
}
.cardRight .star-block, .study-plan-block  {
    margin: 0px 0px 0px auto; display: flex; 
    align-items: center;
    color: #009999;
    transition: opacity .3s linear;
    cursor: pointer;
    font-size: 15px;
}
.cardRight .star-block:hover, .study-plan-block:hover {
    opacity: 60%;
}

.pId {
    font-size: 20px;
    margin-bottom: 5px;
}

.pName {
    font-size: 28px;
    font-weight: bold;
    padding-bottom: 10px;
    margin-bottom: 6px;
}

.source,
.difficulty {
    color: #696666;
    font-size: 15px;
}


.sampleBox {
    border-radius: 5px;
    border: 1px solid #3D3737;
    margin-top: 10px;
    margin-bottom: 10px;
    overflow: hidden;
}

.sample {
    background-color: #EEEEEE;
    padding: 5px;
    white-space: pre-wrap;

}

.el-divider {
    width: 100%;
    margin: 15px 0;
}

.node {
    font-size: 20px;
    font-weight: bold;
    margin-bottom: 10px;
}

.node~div {
    margin-left: 10px;
    margin-right: 10px;
}

.node::before {
    content: "";
    display: inline-block;
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background-color: #3F3C3C;
    margin-right: 8px;
    vertical-align: middle;
}

.inputs {
    border-bottom: 1px solid #3D3737;
    padding: 5px;
    display: flex;
    justify-content: space-between;
    padding-right: 10px;
}

.outputs {
    border-bottom: 1px solid #3D3737;
    border-top: 1px solid #3D3737;
    padding: 5px;
    display: flex;
    justify-content: space-between;
    padding-right: 10px;
}

.el-tag {
    /* background-color: #EAEAEA; */
    border-radius: 8px;
    padding: 4px 10px 4px 10px;
    margin: 4px 4px 4px 4px;
    color: black;
}
.el-dialog {
    display: flex;
    flex-direction: column;
}
.el-button {
    justify-content: flex-end;
    margin-top: 20px;
    margin-left: 260px;
    display: flex;
    padding-left: 20px;
    padding-right: 20px;
}
.el-checkbox-group {
    display: flex;
    flex-direction: column;
    padding-left: 20px;
}
</style>