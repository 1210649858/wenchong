<template>
    <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>广告管理</el-breadcrumb-item>
            <el-breadcrumb-item>广告列表</el-breadcrumb-item>
        </el-breadcrumb>
        <div class="hengxian"></div>

        <el-form :inline="true" :model="search" class="search" ref="ruleForm">
            <el-form-item>
                <el-button type="success" size="small" @click="advertCreate">新增广告</el-button>
            </el-form-item>
            <el-form-item label="所属分类" prop="advert_node_id">
                <el-select size="small" v-model="search.advert_node_id" filterable placeholder="请选择">
                    <el-option
                            v-for="item in advert_nodes"
                            :key="item.id"
                            :label="item.name"
                            :value="item.id">
                    </el-option>
                </el-select>
            </el-form-item>
            <el-form-item label="广告名称" prop="keyword">
                <el-input size="small" v-model="search.keyword"></el-input>
            </el-form-item>
            <el-form-item>
                <el-button size="small" type="warning" @click="searchname" icon="el-icon-search">搜 索</el-button>
                <el-button size="small" type="info" @click="resetForm('ruleForm')" icon="el-icon-refresh">重 置</el-button>
            </el-form-item>
        </el-form>

        <el-table :data="adverts" style="width: 100%">
            <el-table-column label="编号" prop="id" width="50"></el-table-column>
            <el-table-column label="广告图片" width="150">
                <template slot-scope="scope">
                    <a :href="scope.row.url" target="_blank"><img :src="scope.row.photo.image"
                                                                  style="width: 60px;height: 60px"></a>
                </template>
            </el-table-column>
            <el-table-column label="广告名称" prop="name" width="200"></el-table-column>
            <el-table-column label="所属分类" prop="advert_node.name" width="150"></el-table-column>
            <el-table-column label="排序" prop="sort" width="80">
                <template slot-scope="scope">
                    <el-input v-model="scope.row.sort"  @change="change(scope.$index, scope.row)" size="small"></el-input>
                </template>
            </el-table-column>

            <el-table-column label="创建时间" width="200">
                <template slot-scope="scope">
                    <i class="el-icon-time"></i><span style="margin-left: 10px">{{ scope.row.created_at | edit_date}}</span>
                </template>
            </el-table-column>
            <el-table-column label="操作" width="200">
                <template slot-scope="scope">
                    <router-link :to="{ name:'advertEdit',params: { id: scope.row.id }}">
                        <el-button size="mini" type="primary">编辑</el-button>
                    </router-link>
                    <el-button size="mini" type="danger" @click="advertDelete(scope.$index, scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <div class="block el-pagination" style="float: left;margin-top: 10px;">
            <span class="el-pagination__total" style="margin-left: 20%;">共 {{page.total}} 条数据</span>
            <!--分页-->
            <el-pagination background layout="prev, pager, next"
                           :total="page.total" :page-size="page.size"
                           @current-change="changePage" :current-page.sync="page.currentPage"
                           style="margin-top:-30px;margin-left:300%">
            </el-pagination>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                adverts: [],
                advert_nodes: [],
                search: {
                    advert_node_id: '',
                    keyword: '',
                },
                page: {
                    total: 0,
                    size: 0,
                    currentPage: 1,
                    num: 1
                }
            }
        },
        created() { //构造函数
            this.init()
            //所有分类
            axios.get(`/admin/advert_nodes`)
                .then((res) => {
                    console.log(res)
                    this.advert_nodes = res.data;
                })
        },
        filters:{
            edit_date:function (val) {
                return val.substring(0,10)
            }
        },
        methods: {
            init() {
                axios.get(`/admin/adverts?page=${this.page.num}&keyword=${this.search.keyword}&advert_node_id=${this.search.advert_node_id}`)
                    .then((res) => {
                        //console.log(res)
                        this.adverts = res.data.data;
                        this.page.total = res.data.total;
                        this.page.size = res.data.per_page;
                    })
            },
            advertCreate() {
                this.$router.push({name: 'advertCreate'})
            },
            advertDelete(index, row) {
                this.$confirm('亲 (●ﾟωﾟ●)确定要删除吗?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    axios.delete(`/admin/adverts/${row.id}`)
                        .then((res) => {
                            this.$message({
                                type: 'success',
                                message: 'OK,删除成功!'
                            });
                            this.init();
                        });
                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });
                });
            },
            //排序
            change(index, row) {
                axios.patch(`/admin/advert/change_sort`,row)
                    .then(() => {
                        this.$message({
                            type: 'success',
                            message: '排序成功!'
                        });
                        this.init();
                    });
            },
            //选择页码
            changePage(val) {
                this.page.num = val;
                this.init()
            },
            //搜索
            searchname() {
                this.init();
            },
            //重置
            resetForm(formName) {
                this.$refs[formName].resetFields();
                this.init();
            },
        }
    }
</script>

<style>
    .search {
        margin-top: 10px;
    }

    .hengxian {
        margin-top: 10px;
        border-top: 1px solid #eeeeee;
    }

    .el-icon-circle-check-outline {
        color: green;
    }

    .el-icon-circle-close-outline {
        color: red;
    }

    .el-icon-circle-check-outline, .el-icon-circle-close-outline {
        cursor: pointer;
        font-weight: bold;
        font-size: 160%;

    }
</style>