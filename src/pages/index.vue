<template>   
    <div id="content">
        <div id='loading-content' v-if="!loaded"></div>
        <!-- <today/> -->
        <div id="change-bg" :class="[isChangeBg ? 'active' : '']" @click="changeBg">
                <transition name="change-bg-button-ani">
                    <svg v-if="!isChangeBg" t="1596732240856" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="1787" width="200" height="200"><path d="M981.448462 133.180788a35.367512 35.367512 0 0 0-35.367512 35.367512v85.103076a505.092283 505.092283 0 0 0-939.449541 221.046951 35.367512 35.367512 0 0 0 32.604425 38.130599 35.367512 35.367512 0 0 0 35.367512-32.604425 434.357258 434.357258 0 0 1 819.53157-165.785213h-93.944954a35.367512 35.367512 0 1 0 0 71.287641h181.2585a35.367512 35.367512 0 0 0 35.367512-35.367512V168.5483a35.367512 35.367512 0 0 0-35.367512-35.367512z m0 379.095521a35.367512 35.367512 0 0 0-38.130599 32.604425 434.357258 434.357258 0 0 1-819.531571 165.785213h100.023746a35.367512 35.367512 0 1 0 0-71.287642H42.551538a35.367512 35.367512 0 0 0-35.367512 35.367513v181.258499a35.367512 35.367512 0 1 0 71.287642 0v-85.655693a505.092283 505.092283 0 0 0 939.449541-221.046951 35.367512 35.367512 0 0 0-34.814895-37.025364z" fill="#ffffff" p-id="1788"></path></svg>
                </transition>
            </div>
        <transition name='submit-pan-ani'>
            <submit-pan 
                :isMobile='isMobile' 
                v-if="isShowSubmitPan" 
                @exit-submit-pan="exitSubmitPan" 
                @submit-todo="submitTodo"
            />

            <div id="show-submit-pan" 
                @click.stop="isShowSubmitPan = true" 
                v-if="!isShowSubmitPan"
            >
                <svg t="1596622673094" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="1989"><path d="M866.016 476h-318.016V165.984q0-15.008-10.496-25.504T512 129.984q-15.008 0-25.504 10.496t-10.496 25.504v310.016h-312q-14.016 0-24.992 10.496T128 512q0 15.008 11.008 25.504t24.992 10.496h312v312q0 14.016 10.496 24.992T512 896q15.008 0 25.504-11.008t10.496-24.992v-312h318.016q15.008 0 25.504-10.496t10.496-25.504q0-15.008-10.496-25.504t-25.504-10.496z" p-id="1990" fill="#fff"></path></svg>
            </div>
        </transition>
        <div id="main-content">
            <control 
                @finish-select="finishSelect" 
                @clear-all="clearAll"
            />
            <todo-pan v-for="(guide,index) in guides"
                :key="index"
                :lists="guide.list"
                :title="guide.name"
                @change-select="changeSelect" 
                @finish-item="finishItem"
            />
        </div>
    </div>
</template>
<script>
import submitPan from '../components/SubmitPan'
//import today from '../components/today'
import control from '../components/control'
import todoPan from '../components/TodoPan'

export default {
    name: 'index',
    data() {
        return {
            screen:[document.body.clientWidth,document.body.clientHeight],
            isShowSubmitPan: false,
            todoList:[],
            loaded: false,
            backgroundUrl: '',
            isChangeBg: false,
        }
    },
    mounted(){
        this.watchResize();
        this.getTodo();
        this.initFile();
        console.log(this.guides);
    },
    computed:{
        isMobile(){
            return (this.screen[0] <= 764);
        },
        guides(){
            return ([
                {
                    name: 'EXPIRED',
                    list: this.todoList.filter(item=>Date.parse(item.deadline) - Date.parse(new Date().toString()) <= 0)
                },
                {
                    name: 'TODAY',
                    list: this.todoList.filter(item=>{
                            let timeRan = Date.parse(item.deadline) - Date.parse(new Date().toString());
                                if(timeRan > 0 && timeRan <= 86400000){
                                return item;
                                }
                        })
                },
                {
                    name: 'TOMORROW',
                    list: this.todoList.filter(item=>{
                            let timeRan = Date.parse(item.deadline) - Date.parse(new Date().toString());
                            if(timeRan > 86400000 && timeRan <= 172800000){
                                return item;
                            }
                        })
                },
                {
                    name: 'OTHERS',
                    list: this.todoList.filter(item=>{
                            let timeRan = Date.parse(item.deadline) - Date.parse(new Date().toString());
                            if(timeRan > 172800000){
                                return item;
                            }
                        })
                }
            ]);
        },
        todayList(){
            return this.todoList.filter(item=>{
                let timeRan = Date.parse(item.deadline) - Date.parse(new Date().toString());
                if(timeRan > 0 && timeRan <= 86400000){
                    return item;
                }
            });
        },
        tomorrowList(){
            return this.todoList.filter(item=>{
                let timeRan = Date.parse(item.deadline) - Date.parse(new Date().toString());
                if(timeRan > 86400000 && timeRan <= 172800000){
                    return item;
                }
            });
        },
        otherList(){
            return this.todoList.filter(item=>{
                let timeRan = Date.parse(item.deadline) - Date.parse(new Date().toString());
                if(timeRan > 172800000){
                    return item;
                }
            });
        }
    },
    methods:{
        //加载必要文件
        initFile(){
            let promiseAll = [];
            promiseAll[0] = this.setBg(this.bgUrl())
            .then((url)=>{this.isChangeBg = false;document.body.style.backgroundImage = `url(${url})`;})
            .catch(()=>{this.isChangeBg = false;});
            Promise.all(promiseAll).then(()=>{
                this.loaded = true;
            });
        },
        //设置背景
        setBg(url){
            let that = this;
            return new Promise((resolve,reject)=>{
                if(this.screen[0] > 764 && !this.isChangeBg){
                    that.isChangeBg = true;
                    let image = new Image();
                    image.onload = function(){
                        resolve(url);
                    }
                    image.onerror = function(){
                        reject();
                    }
                    image.src = url;
                }else{
                    reject();
                }
            });
        },
        //监听窗口大小变化
        watchResize(){
            let that = this;
            window.onresize = function(){
                that.screen = [document.body.clientWidth,document.body.clientHeight];
            }
        },
        //退出输入todo的内容框
        exitSubmitPan(){
            this.isShowSubmitPan = false;
        },
        //改变选中状态
        changeSelect(flag,id){
            flag = ((flag === undefined) ? true : !flag);
            this.todoList.forEach(element => element.selected = element.id === id ?  flag : element.selected);
        },
        //点击完成选中元素
        finishSelect(){
            this.todoList.forEach(element=>{
                if(element.selected){
                    this.axios.post('deleteTodo',{id: element.id}).then(res=>{
                        res = res.data;
                        if(res.status === 200){
                                this.todoList = res.message;
                            }else{
                                alert(res.message);
                                console.log(res.description);
                                return ;
                            }
                    }).catch(err=>{
                        alert(err.data.message);
                        console.log(err.data.description);
                        return ;
                    });
                }
            });
            //this.todoList = this.todoList.filter(item=>!item.selected);
        },
        //清除全部
        clearAll(){
                this.todoList.forEach(element=>{
                    this.axios.post('deleteTodo',{id: element.id}).then(res=>{
                        res = res.data;
                        if(res.status === 200){
                            this.todoList = res.message;
                        }else{
                            alert(res.message);
                            console.log(res.description);
                            return ;
                        }
                    }).catch(err=>{
                        alert(err.data.message);
                        console.log(err.data.description);
                        return ;
                    });
                });
        },
        //删除特定项
        finishItem(id){
            this.axios.post('deleteTodo',{id}).then(res=>{
                res = res.data;
                if(res.status === 200){
                    this.todoList = res.message;
                }else{
                    alert(res.message);
                    console.log(res.description);
                }
            }).catch(err=>{
                alert(err.data.message);
                console.log(err.data.description);
            });
            //this.todoList = this.todoList.filter(item=>item.id != id);
        },

        //提交todo
        submitTodo(deadline,title,description){
            this.axios.post('submit',{deadline,title,description}).then((res) => {
                res = res.data;
                if(res.status === 200){
                    this.todoList = res.message;
                    this.isShowSubmitPan = false;
                }else{
                    alert(res.message);
                    console.log(res.description);
                }
            }).catch((err) => {
                alert(err.data.message);
                console.log(err.data.description);
            });
        },
        //获取todo
        getTodo(){
            this.axios.post('getList').then(res=>{
                res = res.data;
                if(res.status === 200){
                    this.todoList = res.message;
                }else{
                    alert(res.message);
                    console.log(res.description);
                }
            }).catch(err=>{
                alert(err.data.message);
                console.log(err.data.description);
            });
        },
        //换背景
        changeBg(){
            this.setBg(this.bgUrl())
            .then((url)=>{this.isChangeBg = false;document.body.style.backgroundImage = `url(${url})`;})
            .catch(()=>{this.isChangeBg = false;});
        },
    },
    components:{
        submitPan,
        //today,
        control,
        todoPan
    }
}
</script>

<style lang="stylus">
   html,body{
        width 100%
        height 100%
        -webkit-font-smoothing: antialiased
        font-family sans-serif
        margin 0
        padding 0
        -webkit-tap-highlight-color rgba(0, 0, 0, 0)
        overflow hidden
    }
    body{
      background-color rgba(230,230,230,.7)
      background-position center
      background-size 100% 100%
      background-repeat no-repeat

      @media screen and (max-width: 764px){
        background-image url("") !important;
      }
    }
    
    #content{
        height 100%;
    }

    #loading-content{
        position absolute;
        top 0
        right 0 
        bottom 0
        left 0
        background-color #ffffff
        background-image url('../assets/images/loading.svg');
        background-size 50px auto
        background-position center;
        background-repeat no-repeat
    }

    #change-bg{
        position fixed
        bottom 40px
        right 40px
        display flex
        justify-content center
        align-items center
        width 35px
        height 35px
        border none
        border-radius 20px
        outline none
        opacity .8
        cursor pointer
        background-color $lightBg
        background-position center
        background-repeat no-repeat 
        background-size 100% auto
        transition transform .3s linear 

        svg{
            width 100%;
            height auto;

            path{
                transition fill .3s linear
            }
        }

        &:hover{
            transform scale(1.1) rotate(90deg);

            path{
                fill $strongGreen
            }
        }
    }
    #change-bg.active{
        background-image url('../assets/images/three-dots.svg')
    }

    #show-submit-pan{
        position absolute
        left 10px
        top 10px
        display flex
        align-items center
        justify-content center
        width 40px
        height 40px
        border none 
        border-radius 20px
        background-color $midGreen
        opacity .8
        cursor pointer
        transition opacity .2s linear,transform .2s linear,background-color .2s linear

        &:hover{
            opacity 1
            transform scale(1.1)
            background-color $strongGreen

        }
        @media screen and (min-width: 764px){
            background-color: $lightBg;
        }

        @media screen and (max-width: 764px){
            left unset 
            top unset 
            bottom 10px
            right 10px
        }

        svg{
            width 80%
            height 80%
        }
    }

    #main-content{
        margin 0 auto
        height 100%
        overflow auto

        @media screen  and (min-width: 764px){
            min-width 500px
            max-width 800px   
            background-color rgba(250,250,250,0.8)
        }

        @media screen and (max-width: 764px){
            padding 0 10px
            box-sizing border-box
            width 100%;
        }
    }

    /** animation */
    .submit-pan-ani-enter{
        transform translateX(-300px)

        @media screen and (max-width: 764px){
            transform translateY(100%)
        }
    }

    .submit-pan-ani-enter-active{
        transition transform .3s cubic-bezier(1, 0.05, 1, 1.6)
    }

    .submit-pan-ani-leave-active{
        transition transform .3s cubic-bezier(0.02,-0.28, 0.99, 0.04)
    }

    .submit-pan-ani-leave-to{
        transform translateX(-300px);

        @media screen and (max-width: 764px){
            transform translateY(100%)
        }
    }

    .change-bg-button-ani-enter{
        opacity 0
    }

    .change-bg-button-ani-enter-active{
        transition opacity .3s linear
    }

    .change-bg-button-ani-leave-to{
        opacity 0
    }

    .change-bg-button-ani-leave-active{
        transition opacity .3s linear
    }


</style>