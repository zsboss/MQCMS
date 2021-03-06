<template>
    <div>
        <!-- 搜索 -->
        <search-form
            ref="searchForm"
            :show-multi-action="false"
            :base-search-form="baseSearchForm"
            :advanced-search-form="advancedSearchForm"
            :show-advanced="false"
            @on-create-form="handleOpenUpdateCreate"
            @on-search="searchData"
            @on-reset="getData"
            @on-multi-del="handleMultiDel"
            @on-export="handleExport"
        />
        <!-- 列表 -->
        <Table
            border
            ref="table"
            :columns="columns"
            :data="limitData"
            :loading="loading"
            class="ivu-mt"
            @on-sort-change="handleSortChange"
            @on-filter-change="handleFilterChange"
            @on-select="handleSelect"
            @on-select-cancel="handleSelectCancel"
            @on-select-all="handleSelectAll"
            @on-select-all-cancel="handleSelectAllCancel"
        >
            <template slot-scope="{ row }" slot="cate_name">
                <span>{{ row.cate_info ? row.cate_info.cate_name : '' }}</span>
            </template>
            <template slot-scope="{ row }" slot="is_publish">
                <Tag v-if="row.is_publish == 1" color="success">已发布</Tag>
                <Tag v-else-if="row.is_publish == 0" color="default">草稿</Tag>
            </template>
            <template slot-scope="{ row }" slot="is_recommend">
                <Tag v-if="row.is_recommend == 1" color="error">推荐</Tag>
                <Tag v-else-if="row.is_recommend == 0" color="default">正常</Tag>
            </template>
            <template slot-scope="{ row }" slot="status">
                <Badge v-if="row.status == 0" status="default" text="禁用" />
                <Badge v-if="row.status == 1" status="processing" text="正常" />
            </template>
            <template slot-scope="{ row }" slot="created_at">
                {{ row.created_at }}
            </template>
            <template slot-scope="{ row, index }" slot="action">
                <a @click="handleOpenUpdateCreate(true, row.id)">编辑</a>
                <Divider type="vertical" />
                <a v-color="'#ed4014'" @click="handleDelete(index)">删除</a>
            </template>
        </Table>
        <div class="ivu-mt ivu-text-center">
            <Page :total="total" show-total :current.sync="current" @on-change="handleChange"/>
        </div>
        <!-- 详情 编辑 新增 -->
        <create-form ref="createForm" @on-create-form="handleOpenUpdateCreate" @on-ok="getData" />
    </div>
</template>
<script>
    import { PostSearch, PostIndex, PostDelete } from '@api/post';
    import CreateForm from './create-form';
    import SearchForm from '@/components/searchform/index';
    
    export default {
        components: {
            CreateForm,
            SearchForm
        },
        data () {
            return {
                columns: [
                    {
                        type: 'selection',
                        width: 60,
                        align: 'center'
                    },
                    {
                        title: 'ID',
                        key: 'id',
                        minWidth: 80
                    },
                    {
                        title: '标题',
                        key: 'post_title',
                        minWidth: 100
                    },
                    {
                        title: '分类',
                        key: 'cate_name',
                        slot: 'cate_name',
                        minWidth: 100
                    },
                    {
                        title: '是否发布',
                        slot: 'is_publish',
                        key: 'is_publish',
                        minWidth: 100
                    },
                    {
                        title: '是否推荐',
                        slot: 'is_recommend',
                        key: 'is_recommend',
                        minWidth: 100
                    },
                    {
                        title: '状态',
                        slot: 'status',
                        minWidth: 100,
                        filters: [
                            {
                                label: '禁用',
                                value: 0
                            },
                            {
                                label: '正常',
                                value: 1
                            }
                        ],
                        filterMultiple: true,
                        filterRemote (value) {
                            // 切换过滤条件时，将条件保存到本地
                            this.filterType = value;
                        }
                    },
                    {
                        title: '发布时间',
                        key: 'created_at',
                        minWidth: 140,
                        sortable: 'custom'
                    },
                    {
                        title: '更新时间',
                        key: 'updated_at',
                        minWidth: 140,
                        sortable: 'custom'
                    },
                    {
                        title: '操作',
                        slot: 'action',
                        align: 'center',
                        minWidth: 140,
                        fixed: 'right'
                    }
                ],
                loading: false,
                list: [],
                selectedData: [],
                currentItem: {},
                current: 1,
                total: 0,
                size: 10,
                sortType: 'normal', // 当前排序类型，可选值为：normal（默认） || asc（升序）|| desc（降序）,
                sortColumns: '',
                filterType: undefined,
                searchForm: {},
                baseSearchForm: {
                    type: 'id',
                    keyword: '',
                    options: [
                        {
                            name: 'ID',
                            value: 'id'
                        },
                        {
                            name: '标题',
                            value: 'post_title'
                        }
                    ]
                },
                advancedSearchForm: [
                    {
                        label_name: '资源类型：',
                        label_prop: 'label_type',
                        ele_value: '',
                        ele_type: 'select',
                        options: [
                            {
                                value: '1',
                                name: '单图'
                            },
                            {
                                value: '2',
                                name: '视频'
                            },
                            {
                                value: '3',
                                name: '其他'
                            }
                        ],
                    },
                    {
                        label_name: '是否发布：',
                        label_prop: 'is_publish',
                        ele_value: '',
                        ele_type: 'select',
                        options: [
                            {
                                value: '1',
                                name: '是'
                            },
                            {
                                value: '0',
                                name: '否'
                            }
                        ],
                    },
                    {
                        label_name: '是否推荐：',
                        label_prop: 'is_recommend',
                        ele_value: '',
                        ele_type: 'select',
                        options: [
                            {
                                value: '1',
                                name: '是'
                            },
                            {
                                value: '0',
                                name: '否'
                            }
                        ],
                    },
                    {
                        label_name: '创建时间：',
                        label_prop: 'created_at',
                        ele_value: '',
                        ele_type: 'daterange',
                        options: {}
                    },
                    {
                        label_name: '更新时间：',
                        label_prop: 'updated_at',
                        ele_value: '',
                        ele_type: 'daterange',
                        options: {}
                    }
                ]
            }
        },
        computed: {
            limitData () {
                let data = [...this.list];

                // 动态计算排序类型
                const sortColumns = this.sortColumns;
                if (this.sortType === 'asc') {
                    data = data.sort((a, b) => {
                        return a[sortColumns] > b[sortColumns] ? 1 : -1;
                    });
                }
                if (this.sortType === 'desc') {
                    data = data.sort((a, b) => {
                        return a[sortColumns] < b[sortColumns] ? 1 : -1;
                    });
                }

                // 动态计算过滤类型
                if (this.filterType && this.filterType.length) {
                    data = data.filter(item => {
                        return this.filterType.indexOf(item.status) >= 0;
                    });
                }

                // 判断是否有选中的数据
                const selectedNames = this.selectedData.map(item => item.id);
                data.map(item => {
                    item._checked = selectedNames.indexOf(item.id) >= 0;
                    return item;
                });

                return data;
            },
            // 因为要动态计算总页数，所以还需要一个计算属性来返回最终给 Table 的 data
            dataWithPage () {
                const data = this.limitData;
                const start = this.current * this.size - this.size;
                const end = start + this.size;
                return [...data].slice(start, end);
            }
        },
        methods: {
            getData () {
                this.loading = true;
                PostIndex({
                    page: this.current,
                    limit: this.size
                }).then(async res => {
                    this.list = res.data;
                    this.total = res.total;
                }).finally(() => {
                    this.loading = false;
                });
            },
            searchData (searchForm) {
                this.searchForm = searchForm;
                this.loading = true;
                PostSearch({
                    page: this.current,
                    limit: this.size,
                    search: searchForm
                }).then(async res => {
                    this.list = res.data;
                    this.total = res.total;
                }).finally(() => {
                    this.loading = false;
                });
            },
            // 点击排序按钮时触发
            handleSortChange ({ key, order }) {
                // 将排序保存到数据
                this.sortColumns = key;
                this.sortType = order;
                this.current = 1;
            },
            // 过滤条件改变时触发
            handleFilterChange () {
                // 从第一页开始
                this.current = 1;
            },
            // 选中一项，将数据添加至已选项中
            handleSelect (selection, row) {
                this.selectedData.push(row);
            },
            // 取消选中一项，将取消的数据从已选项中删除
            handleSelectCancel (selection, row) {
                const index = this.selectedData.findIndex(item => item.id === row.id);
                this.selectedData.splice(index, 1);
            },
            handleSelectAll (selection) {
                selection.forEach(item => {
                    if (this.selectedData.findIndex(i => i.id === item.id) < 0) {
                        this.selectedData.push(item);
                    }
                });
            },
            // 取消当前页全选时，将当前页的数据（即 dataWithPage）从已选项中删除
            handleSelectAllCancel () {
                const selection = this.dataWithPage;
                selection.forEach(item => {
                    const index = this.selectedData.findIndex(i => i.id === item.id);
                    if (index >= 0) {
                        this.selectedData.splice(index, 1);
                    }
                });
            },
            // 清空所有已选项
            handleClearSelect () {
                this.selectedData = [];
            },
            // 编辑创建数据
            handleOpenUpdateCreate (status, updateIndex) {
                if (updateIndex < 0) {
                    this.$router.push('/portal/post/create');
                } else {
                    this.$refs.createForm.handleShowUpdateCreate(status, updateIndex);
                }
            },
            handleDelete (index) {
                this.updateIndex = index;
                this.$Modal.confirm({
                    title: '删除提示',
                    content: '确定删除该条记录吗？',
                    onOk: () => {
                        PostDelete({
                            id: this.list[index].id
                        }).then(res => {
                            this.$Message.success('删除成功');
                            this.current = 1;
                            this.getData();
                        }).finally(() => {
                        });
                    }
                });
            },
            handleMultiDel () {
                if (this.selectedData.length === 0) {
                    this.$Message.error('请选择至少一个元素');
                    return false;
                }
                const ids = this.selectedData.map(item => item.id);
                this.$Modal.confirm({
                    title: '删除提示',
                    content: '确定要批量删除吗？',
                    onOk: () => {
                        this.$Message.success('删除成功: ' + ids);
                    }
                });
            },
            handleChange (page) {
                this.current = page;
                if (this.searchForm) {
                    this.searchData(this.searchForm);
                } else {
                    this.getData();
                }
            },
            handleExport () {
                this.$Message.success('导出成功');
            }
        }
    }
</script>
