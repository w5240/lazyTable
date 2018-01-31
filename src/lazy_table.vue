<template>
    <div class="lazyTable">
        <div class="common left" :style="{width:`calc(${cellWidth} * ${intersection.length})`}">
            <div class="head" ref="headLeft">
                <ul :style="{height:cellHeight}"><li v-for="e in intersection" :style="{height:cellHeight,width:cellWidth}">{{e}}</li></ul>
            </div>
            <div class="content" :style="{height:`calc(100% - ${cellHeight} - 8px)`}">
                <!--:style="{backgroundColor:whichHover===index?'#f6f6f6':''}"-->
                <ul :style="{height:cellHeight}" v-for="(e,index) in leftHeader">
                    <li :style="{height:cellHeight,width:cellWidth}" v-for="cer in e">{{cer}}</li>
                </ul>
            </div>
        </div>
        <div class="common right" :style="{width:`calc(100% - (${cellWidth} * ${intersection.length}))`}">
            <div class="head" ref="headRight">
                <ul :style="{width:`calc(${cellWidth} * ${header.length})`,height:cellHeight}">
                    <li :style="{height:cellHeight,width:cellWidth}" v-for="e in header">{{e}}</li>
                </ul>
            </div>
            <div class="content" @scroll='divScroll($event)' ref="scrollDiv" :style="{height:`calc(100% - ${cellHeight})`}">
                <div :style="{height:`calc(${cellHeight} * ${rows.length})`}">
                    <ul ref="emptyLine" :style="{width:`calc(${cellWidth} * ${header.length})`}"></ul>
                    <!--@mouseenter="whichHover=showRowsIndex[index]" @mouseleave ="whichHover=''"-->
                    <ul class="" ref="contentUl" :style="{width:`calc(${cellWidth} * ${header.length})`,height:cellHeight}" v-for="(e,index) in showRows">
                        <li ref="emptyRow" :style="{height:cellHeight}"></li>
                        <li :style="{height:cellHeight,width:cellWidth}" v-for="cer in e">{{cer}}</li>
                    </ul>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
    export default{
        data(){
            return{
                intersection:this.left[0],  //左上重合部分
                leftHeader:this.left.slice(1),  //左下header部分

                showRows:[],  //当前展示数据
//                showRowsIndex:[],
                showWidthLength:'',  //横向展示个数
                showHeightLength:'',  //竖向展示个数
                scrollLeft:'',
                scrollTop:'',
//                whichHover:'',

                widthBegin:'',
                heightBegin:'',

            }
        },
        props:{
            left:{
                type:Array,
                required:true,  //左边数据
            },
            header:{
                type:Array,
                required:true,  //顶部header
            },
            rows:{
                type:Array,
                required:true,  //右边数据
            },
            headerHeight:{
                type:String,
                default:'65px',
            },
            cellHeight:{
                type:String,
                default:'50px',
            },
            cellWidth:{
                type:String,
                default:'100px',
            },

        },
        methods:{
            init(){
                this.showWidthLength = Math.round(this.$refs.scrollDiv.clientWidth/parseInt(this.cellWidth));
                this.showWidthLength += 2;
                this.showHeightLength = Math.round(this.$refs.scrollDiv.clientHeight/parseInt(this.cellHeight));
                this.showHeightLength += 2;

                console.log(this.showWidthLength);
                console.log(this.showHeightLength);
                this.drawRows(0 , this.showWidthLength , 0 , this.showHeightLength);
            },
            drawRows(widthBegin ,widthEnd , heightBegin ,heightEnd){
//                this.showRowsIndex=[];
                this.showRows = this.rows.filter((e,index)=>{
                    if(index< heightEnd && index>=heightBegin){
//                        this.showRowsIndex.push(index);
                        return e
                    }
                });
                this.showRows = this.showRows.map((e)=>{
                    return e.filter((a,i)=>{
                        if(i < widthEnd && i >= widthBegin){
                            return String(a)
                        }
                    })
                })
//                if(this.showRows.length<this.showHeightLength){
//                    for(let i=0;i<this.showHeightLength-this.showRows.length;i++) {
//                        this.showRows.push([])
//                    }
//                }
            },
            divScroll(e){
                this.scrollLeft = this.$refs.scrollDiv.scrollLeft;
                this.scrollTop = this.$refs.scrollDiv.scrollTop;
            },
        },
        watch:{
            scrollLeft(n){
                //scrollTo QQ浏览器不兼容
//                document.querySelector('.right .head').scrollTo(n,0)
                document.querySelector('.right .head').scrollLeft = n ;
                let width = parseInt(this.scrollLeft/parseInt(this.cellWidth));
                if(this.widthBegin !== width) {
                    this.$refs.emptyRow.forEach((e) => {
                        e.style.width = parseInt(this.cellWidth) * width + 'px';
                    });
                    this.widthBegin = width;
                }
            },
            scrollTop(n){
                document.querySelector('.left .content').scrollTop = n ;
                let height = parseInt(this.scrollTop/parseInt(this.cellHeight));
                if(this.heightBegin !== height) {
                    this.$refs.emptyLine.style.height = parseInt(this.cellHeight) * (height) + 'px';
                    this.heightBegin = height;
                    document.querySelector('.right .content').scrollTop = n ;
                    //判断灰白显示情况
                    if(parseInt(this.scrollTop/parseInt(this.cellHeight))%2 === 0){
                        this.$refs.contentUl.forEach((e) => {
                            e.setAttribute('class', '')
                        })
                    }else{
                        this.$refs.contentUl.forEach((e) => {
                            e.setAttribute('class', 'change')
                        })
                    }
                }
            },
            heightBegin(n,o){
                if(n !== o)
                    this.drawRows(this.widthBegin , this.widthBegin+this.showWidthLength, n , n+this.showHeightLength)
            },
            widthBegin(n,o){
                if(n !== o)
                    this.drawRows(n , n+this.showWidthLength, this.heightBegin , this.heightBegin+this.showHeightLength)
            }
        },
        mounted(){
            this.$nextTick(()=>{
                this.init();
            })
        }
    }

</script>
<style lang="scss" type="text/scss" rel="stylesheet/scss">
    .lazyTable{
        height:100%;
        width:100%;
        border:1px solid #e0e0e0;

        .left{
            float: left;
            height:100%;
            ul{
                &:after{
                    content: '';
                    clear:both;
                    height: 0;
                    visibility: hidden;
                }
                li{
                    float: left;
                    line-height:50px;
                    text-align: center;
                }
            }
            .head{
                width:100%;
                border-bottom:1px solid #e0e0e0;
                border-right:1px solid #e0e0e0;
            }
            .content{
                position: relative;
                z-index: 10;
                width:100%;
                overflow: hidden;
                border-right:1px solid #e0e0e0;

                ul{
                    box-sizing: border-box;
                    border-bottom:1px solid #e0e0e0;
                    &:nth-of-type(2n){
                        background-color: #fff;
                    }
                    &:nth-of-type(2n+1){
                        background-color: #fafafa;
                    }
                }
            }

        }

        .right{
            float: right;
            height:100%;
            ul{
                &:after{
                    content: '';
                    clear:both;
                    height: 0;
                    visibility: hidden;
                }
                li{
                    float: left;
                    line-height:50px;
                    text-align: center;
                }
            }
            .head{
                width:calc(100% - 8px);
                border-bottom:1px solid #e0e0e0;
                overflow: hidden;
            }
            .content{
                width:100%;
                overflow-x: auto;
                overflow-y: auto;

                ul{
                    box-sizing: border-box;
                    border-bottom:1px solid #e0e0e0;
                    &:first-child{
                        border:0;
                    }
                    &:nth-of-type(2n+1){
                        background-color: #fff;
                    }
                    &:nth-of-type(2n){
                        background-color: #fafafa;
                    }
                    &.change:nth-of-type(2n){
                        background-color: #fff;
                    }
                    &.change:nth-of-type(2n+1){
                        background-color: #fafafa;
                    }
                    /*&:hover{ background-color: #f6f6f6;}*/
                    /*&.change:hover{ background-color: #f6f6f6;}*/
                }
            }
        }


    }

</style>