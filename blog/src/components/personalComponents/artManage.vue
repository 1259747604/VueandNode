<template>
    <div class="artManage">
        <div class="select">
            <span>文章类型:</span>
            <Select v-model="selectV" style="width:200px" @on-change="select">
                <Option v-for="(item,index) in artType" :value="item.type" :key="index">{{ item.type }}</Option>
            </Select>
        </div>
        <router-view/>
    </div>
</template>

<script>
    export default {
        name: "Manage",
        data () {
            return {
                artType: [],
                selectV:'',
            }
        },
        created(){
            /*获取类型*/
            this.getType();
        },
        methods:{
            getType(){
                this.$axios.get('/artType')
                    .then(data => {
                        if(data.data.keepStatus === 408){
                            return this.$store.commit('changeSessionIsNew',true);
                        }
                        const status = data.data.status;
                        if(status){
                            this.artType = data.data.data;
                        }
                    })
                    .catch(err => {
                        console.log(err);
                    });
            },
            select(v){
                const data = this.artType.filter(item => {
                    return item.type === v;
                });
                this.$router.push({name:'artListType',params:{id:data[0]._id}});
            }
        }
    }
</script>

<style scoped>
    .artManage{
        padding: 20px;
    }
    .select {
        margin-bottom: 30px;
    }
    .select span{
        display: inline-block;
        height: inherit;
        font-size: 20px;
        font-family: 华文楷体;
        vertical-align: middle;
    }
</style>