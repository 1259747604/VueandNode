<template>
    <div class="artList">
        <Table border :columns="columns" :data="data"></Table>
        <Page :total="total" show-elevator @on-change="change"/>
    </div>
</template>

<script>
    export default {
        name: "selectList",
        data () {
            return {
                columns: [
                    {
                        title: 'ID',
                        key: '_id',
                    },
                    {
                        title: 'Title',
                        key: 'title'
                    },
                    {
                        title: 'Type',
                        key: 'type'
                    },
                    {
                        title: 'CreatedTime',
                        key: 'created'
                    },
                    {
                        title: 'UpdateTime',
                        key: 'updatedAt'
                    },
                    {
                        title: 'Action',
                        key: 'action',
                        width: 150,
                        align: 'center',
                        render: (h, params) => {
                            return h('div', [
                                h('Button', {
                                    props: {
                                        type: 'primary',
                                        size: 'small'
                                    },
                                    style: {
                                        marginRight: '5px'
                                    },
                                    on: {
                                        click: () => {
                                            this.editor(params.row)
                                        }
                                    }
                                }, 'Editor'),
                                h('Button', {
                                    props: {
                                        type: 'error',
                                        size: 'small'
                                    },
                                    on: {
                                        click: () => {
                                            this.remove(params.row)
                                        }
                                    }
                                }, 'Delete')
                            ]);
                        }
                    }
                ],
                initData:[],
                data: [],
                id: '',
                total:0
            }
        },
        created(){
            this.getData(this.$route.params.id);
        },
        beforeRouteUpdate (to, from, next) {
            const id = to.params.id;
            this.getData(id);
            next();
        },
        methods:{
            getData(id){
                /*获取数据*/
                this.$axios.get(`/selectList/${id}`)
                    .then(data => {
                        const status = data.data.status;
                        if(status){
                            /*得到初始数据*/
                            const list = data.data.data;
                            this.initData = list;//存到一个可能以后会用到的数组里
                            this.id = id;//类型id
                            this.total = data.data.total;//得到总页码

                            this.$parent.selectV = data.data.data[0].type.type || '';
                            this.filterData(list);
                            // this.$router.push(`/personal/artManage/artListType/${id}`);
                        }
                    })
                    .catch(err => {
                        console.log(err);
                    })
            },
            filterData(list){
                /*对初始数据进行处理*/
                let arr = list.map(item => {
                    item.typeId = item.type._id;
                    item.type = item.type.type;
                    item.created = new Date(item.created).toLocaleDateString();
                    item.updatedAt = new Date(item.updatedAt).toLocaleDateString();
                    return item;
                });
                this.data = arr;
            },
            editor(data){
                const id = data._id;
                this.$router.push({name:'updateArt',params:{id}});
            },
            remove (data) {
                const _this = this;

                this.$Modal.confirm({
                    title:`删除${data.title}`,
                    content:'你可想好了 删了可就没了',
                    onOk(){
                        const id = data._id;
                        _this.$axios({
                            method:'delete',
                            url:'/article',
                            data:{id,typeId:data.typeId}
                        })
                            .then(data => {
                                if(data.data.keepStatus === 408){
                                    return _this.$store.commit('changeSessionIsNew',true);
                                }
                                const status = data.data.status;
                                if(status){
                                    _this.$Notice.success({
                                        title:'Success',
                                        desc:'你删除了这篇文章',
                                        duration:2,
                                        onClose(){
                                            _this.getData(_this.$route.params.id);
                                        }
                                    });
                                }
                                else{
                                    _this.$Notice.success({
                                        title:'Error',
                                        desc:'外星人破坏 文章没删成',
                                        duration:2,
                                    });
                                }
                            })
                            .catch(err => {
                                console.log(err);
                            });
                    }
                });
            },
            change(index){
                this.$axios.get(`/selectList/${this.id}/${index}`)
                    .then(data => {
                        const status = data.data.status;
                        if(status){
                            /*得到初始数据*/
                            const list = data.data.data;
                            this.initData = list;//存到一个可能以后会用到的数组里
                            this.filterData(list);
                        }
                    })
                    .catch(err => {console.log(err)})
            }
        }
    }
</script>

<style scoped>

</style>