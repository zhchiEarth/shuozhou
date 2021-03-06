<template>
    <el-form :model="cases" :rules="Rules" ref="cases" label-width="100px">
        <el-form-item label="分类" prop="category_id">
            <el-select v-model="cases.category_id" placeholder="请选择">
                <el-option
                        v-for="item in categories"
                        :label="item.name"
                        :key="item.id"
                        :value="item.id">
                </el-option>
            </el-select>
        </el-form-item>
        <el-form-item label="标题" prop="title">
            <el-input v-model="cases.title" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="副标题" prop="subtitle">
            <el-input v-model="cases.subtitle" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="页面图像" prop="page_image">
            <div class="picture-uploader">
                <img v-if="cases.page_image" width="80" height="80" :src="cases.page_image" class="avatar" @click="imageVisible = true">
                <div v-else class="el-upload el-upload--text" @click="imageVisible = true">
                    <i class="el-icon-plus picture-uploader-icon"></i>
                </div>
            </div>
        </el-form-item>
        <el-form-item label="内容" prop="content">
            <Tinymce :height=400 ref="editor" v-model="cases.content"></Tinymce>
        </el-form-item>
        <el-form-item label="发布时间" prop="published_at">
            <el-date-picker
                    v-model="cases.published_at"
                    type="datetime"
                    format="yyyy-MM-dd HH:mm:ss"
                    placeholder="选择日期时间">
            </el-date-picker>
        </el-form-item>
        <el-form-item label="是否草稿？" prop="is_draft">
            <el-radio class="radio" v-model="cases.is_draft" :label="0">是</el-radio>
            <el-radio class="radio" v-model="cases.is_draft" :label="1">否</el-radio>
        </el-form-item>

        <el-form-item>
            <el-button @click="handleCancel">取消</el-button>
            <el-button type="primary" @click="onSubmit" :loading="addLoading">提交</el-button>
        </el-form-item>
        <select-images :max="1" :visible="imageVisible" @close="imageVisible = false" @submit="selectImagesSubmit"></select-images>
    </el-form>
</template>

<script>

import { parseTime } from '@/utils';
import { getList, addCase, updateCase, destroyCase } from 'api/cases';
import FormMixin from './FormMixin'
import Tinymce from '@/components/Tinymce'
import SelectImages from "@/components/Attachment/selectImages";

export default {
//    name:'NewForm',
    mixins: [FormMixin],
    components: { Tinymce, SelectImages },
    props: {
        cases: {
            type: Object,
            default() {
                return {
                    page_image: '',
                    published_at: '',
                    is_draft: 1
                }
            }
        }
    },
    data() {
        return {
            category:{},
            categories: [],
            imageVisible: false,
            isDisabled: false,
            addLoading: false,
            Rules: {
                title: [
                    {required: true, message: '请输入主标题', trigger: 'blur'},
                ],
                subtitle: [
                    {required: true, message: '请输入副标题', trigger: 'blur'},
                ],
                content: [
                    {required: true, message: '请输入内容', trigger: 'blur'}
                ],
                page_image: [
                    {required: true, message: '请选择图片', trigger: 'blur'}
                ],
//                published_at: [
//                    {type: 'date', required: true, message: '选择发布时间', trigger: 'change'}
//                ],
            },
        }
    },
    methods: {
        onSubmit(){
            if (!this.cases.published_at || this.cases.published_at == '0-0-0 0:0:0') {
                this.$message({
                    message: '请选择发布时间！',
                    type: 'error'
                });
                return false;
        }
            this.$refs.cases.validate((valid) => {
                if (valid) {
                    this.cases.published_at = parseTime(this.cases.published_at, '{y}-{m}-{d} {h}:{i}:{s}');

                    this.addLoading = true
                    if (this.cases.id == undefined) {
                        addCase(this.cases).then(response => {
                            this.$message({
                                message: '添加成功！',
                                type: 'success'
                            });
                            this.handleCancel()
                        })
                    } else {
                        updateCase(this.cases.id, this.cases).then(response => {
                            this.$message({
                                message: '更新成功！',
                                type: 'success'
                            });
                            this.handleCancel()
                        })
                    }
                }
            });
        },
        handleCancel(){
            this.$router.push({path: "/case"})
        },
        //选择图片
        selectImagesSubmit(images) {
            this.cases.page_image = images[0].links
        }
    }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
    @import "src/styles/mixin.scss";
    .editor-container {
        min-height: 500px;
        margin: 0 0 30px;
        .editor-upload-btn-container {
            text-align: right;
            margin-right: 10px;
            .editor-upload-btn {
                display: inline-block;
            }
        }
    }

    .image-preview {
        width: 200px;
        height: 200px;
        position: relative;
        border: 1px dashed #d9d9d9;
        float: left;
        /*margin-left: 50px;*/
        .image-preview-wrapper {
            position: relative;
            width: 100%;
            height: 100%;
            img {
                width: 100%;
                height: 100%;
            }
        }
        .image-preview-action {
            position: absolute;
            width: 100%;
            height: 100%;
            left: 0;
            top: 0;
            cursor: default;
            text-align: center;
            color: #fff;
            opacity: 0;
            font-size: 20px;
            background-color: rgba(0, 0, 0, .5);
            transition: opacity .3s;
            cursor: pointer;
            text-align: center;
            line-height: 200px;
            .el-icon-delete {
                font-size: 36px;
            }
        }
        &:hover {
            .image-preview-action {
                opacity: 1;
            }
        }
    }
    .image-app-preview {
        width: 260px;
        height: 160px;
        position: relative;
        border: 1px dashed #d9d9d9;
        float: left;
        /*margin-left: 50px;*/
        .app-fake-conver {
            height: 44px;
            position: absolute;
            width: 100%; // background: rgba(0, 0, 0, .1);
            text-align: center;
            line-height: 64px;
            color: #fff;
        }
    }
</style>