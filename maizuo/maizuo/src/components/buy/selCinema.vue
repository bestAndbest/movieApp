<template>
    <div class="selcinema">
       <ul class="header">
           <li class="iconfont icon-arrow1"></li>
           <li>{{movieName}}</li>
           <li class="iconfont icon-search"></li>
       </ul>
        <div class="swiper2">
            <swiper :options="swiperOption2"  >
                <swiper-slide v-for="(ele,index) in showCinemas">
                    <div @click="clickHandler(ele.cinemaList,index)" :class="{'color':selected===index}">
                       {{$moment(ele.showDate*1000).format('MM月DD日')}}
                        <p :class="{'bb':selected===index}"></p>
                    </div>
                </swiper-slide>
            </swiper>
        </div>
        <ul  class="ticketList"  ref="TL" >
            <!--<li v-for="ele in schedules">-->
                <!--<dl class="first_dl">-->
                    <!--<dt>{{$moment(ele.showAt*1000).format('HH:mm')}}</dt>-->
                    <!--<dd>{{$moment(ele.endAt*1000).format('HH:mm')}}散场</dd>-->
                <!--</dl>-->
                <!--<dl class="second_dl">-->
                    <!--<dt>{{ele.filmLanguage}}</dt>-->
                    <!--<dd>{{ele.hallName}}</dd>-->
                <!--</dl>-->
                <!--<div class="pribuy">-->
                    <!--<div><b>￥</b><span>{{ele.salePrice/100}}</span></div>-->
                    <!--<div class="buy">购票</div>-->
                <!--</div>-->
            <!--</li>-->
        </ul>
        <!--<div class="without" v-else>-->
            <!--<img src="../../assets/imgs/without.png"/>-->
            <!--<p>暂无场次</p>-->
        <!--</div>-->
    </div>
</template>

<script>
    import {mapState} from 'vuex'
    export default {
        name: "selCinema",
        data(){
            return {
                cinemaExtendList:[],
                showCinemas:[],
                movieName:'',
                cinemas:[],
                tempDate:[],
                swiperOption2:{
                    slidesPerView: 3,
                },
                selected:0,
            }
        },
        computed:{
            ...mapState(['storeData']),
        },
        methods:{
            clickHandler(list,index){
                console.log(list);
                this.cinemas=[]
                for(let i=0;i<this.tempDate.length;i++){
                    for(let j=0;j<list.length;j++){
                        if(this.tempDate[i].cinemaId==list[j]){
                            this.cinemas.push(this.tempDate[i])
                        }
                    }
                }
                console.log(11111,this.cinemas);
                this.selected=index
                // this.$store.dispatch('getDay1Movie',{filmId:this.movieInfo.filmId,cinemaId:this.cinemaId,t:ele})
                if(this.cssFlag){
                    this.$refs.TL.setAttribute('class','ticketList animated fadeInLeft')
                }
                setTimeout(()=>{
                    this.$refs.TL.setAttribute('class','ticketList')
                },800)
            },
            getfilmsData(filmId,cityId){
                let url='https://m.maizuo.com/gateway/?filmId='+filmId+'&cityId='+cityId+'&k=4119745'
                this.$axios({
                    url,
                    method:'get',
                    headers:{
                        'X-Client-Info':'{"a":"3000","ch":"1002","v":"5.0.4","e":"15728699844239132721697","bc":"110100"}',
                        'X-Host':'mall.film-ticket.cinema.film-show-cinema'
                    }
                }).then(res=>{
                    console.log(res);
                    if(res.status===200 && res.data.status===0){

                        let {cinemaExtendList,showCinemas}=res.data.data
                        this.cinemaExtendList=cinemaExtendList
                        this.showCinemas=showCinemas
                        console.log('cinemaExtendList', cinemaExtendList);
                        let list=this.showCinemas[0].cinemaList
                        for(let i=0;i<this.tempDate.length;i++){
                            for(let j=0;j<list.length;j++){
                                if(this.tempDate[i].cinemaId==list[j]){
                                    this.cinemas.push(this.tempDate[i])
                                }
                            }
                        }
                        console.log('this.cinemas',this.cinemas);
                    }
                }).catch(err=>console.log(err))
            },
            //获取当前城市的电影院
            getCinemaData(id){
                let url='https://m.maizuo.com/gateway?cityId='+id+'&ticketFlag=1&k=8921000'
                this.$axios({
                    url,
                    method: 'get',
                    headers: {
                        'X-Client-Info': '{"a":"3000","ch":"1002","v":"5.0.4","e":"15728699844239132721697","bc":"440300"}',
                        'X-Host': 'mall.film-ticket.cinema.list'
                    }
                }).then(res=>{
                    if(res.status===200 && res.data.msg==='ok'){
                        setTimeout(()=>this.isLoading=false,1000)
                        let {cinemas}=res.data.data
                        localStorage.setItem('cinemas',JSON.stringify(cinemas))
                        // this.cinemas=cinemas
                        this.tempDate=cinemas
                        console.log(cinemas);
                    }

                }).catch(err=>{
                    console.log(err);
                })
            },

        },
        created() {
           this.movieName=localStorage.getItem('movieName')
            let filmId=this.$route.params.id
            let city=JSON.parse(localStorage.getItem('city'))
            this.getCinemaData(city.cityId)
            this.getfilmsData(filmId,city.cityId)
            // this.$store.dispatch('getStoreDate',this.cinemaId)
            // console.log('this.showCinemas',this.showCinemas);
        },

    }
</script>

<style lang="scss" scoped>
    .selcinema{
        width: 100%;
        height: 100%;
        .header{
            display: flex;
            box-sizing: border-box;
            padding: 14px;
            justify-content: space-between;
            font-size: 18px;
            .iconfont{
                font-size: 18px;
            }
        }
        .swiper2{
            border-top: 1px solid #F6F6F6;
            background-color: #fff;
            padding-top: 10px;
            .swiper-container{
                text-align: center;
                font-size: 14px;
                .swiper-slide{
                    touch-action:none;
                    .color{
                        color: #FF5F16;
                    }
                    .bb{
                        width: 70%;
                        margin: 0px auto;
                        margin-top: 10px;
                        border-bottom: 2px solid #FF5F16;
                    }
                }
            }
        }
    }
</style>