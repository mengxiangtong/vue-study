### 饿了么 vue 项目总结
> 项目效果预览  [ele效果预览]( https://hugeorange.github.io/demo/ele )

> 项目源码地址 [ele源码](https://github.com/hugeorange/elemedemo)
```
跟着慕课网黄轶老师  敲饿了么 vue 项目
```
[作者项目源代码地址](https://github.com/ustbhuangyi/vue-sell)

### 项目完成之后 npm run build
    这本来是写在最后面一段的，我现在把他写在了最前面，方便我们事先知道，整个项目做完之后是什么样子的
  - 项目完成之后在 根目录 下 npm run build （就是 npm run dev 的那个目录）
  - 会在根目录下生成一个 dist 目录，其中包含着 index.html 和一个css目录，一个js目录
  - 按官方说，这个 dist 目录必须 http server 环境下才能运行
  - 下个 xampp 在本地服务下访问
  访问时出现了以下几个问题：
  1. css js 引用路径出错  （将 cofig目录下的 index.js  里的 assetsPublicPath:'./' 这样设置即可）
  2. 由于视频上是写了一个 node 后端服务，访问本地的 data.json 文件，然后用 vue-resource 访问这个 node 服务才请求到的接口
     打包之后，访问不到这个 node 服务了，自然就出错了
     如何解决：我在朋友的帮助下，知道了 easy-mock这个东西，然后 把data.json 文件 用 easy-mock 制作成了 一个 http 接口
     后来因为 github 不能访问 http 接口，又把 http 改成了 https（我最后打包的项目放在了 gitpages上了）

```

```
# 项目启动

  添加静态资源文件，修改 build、dev-serve.js mock模拟数据，
  添加 meta 标签
  碰到 换台机器 报错-没有 modules ，暂时解决方法，删除整个 node_modules,然后重新 npm install

#### 建立好 es6 书写， stylus书写方法，增加了tab导航栏，配置好了路由

	学习了 1px 边框制作（不过感觉用处不大）
	编写 stylus mixin 函数并在引用
	(注意：引入外界stylus样式文件时：只能用 @import 在style标签里引用
	且路径不可以在 webpack.base.conf.js alias别名)

	全局通用样式，字体文件，图标文件
	可以用统一在同级目录下用一个 index.styl
	文件作为出口，在其内部 用 @import './minix.styl' 引入
	然后在再 webpack.base.conf.js  统一配置 alias 别名
	之后再在 main.js  引入这个 index.styl 文件 即可使用这些样式文件
	如：import 'common/stylus/index.styl'

	stylus 文件书写
		1.尽量使用类 css 语法即 {}
		2.尽量避免拷贝代码，产生多余的空格缩进问题

#
	做完之后好好学习一下 flex 布局
	display:flex  flex:1
	完成 header 组件 ，goods组件 完成布局



> #####better-scroll  的用法


    better-scroll 实现列表滚动联动
    1. 初始化 better-scroll
        _initScroll() {
            this.menuScroll = new BScroll(this.$refs.menuWrapper,{
                click:true        //默认派发点击事件
            });

            this.foodsScroll = new BScroll(this.$refs.foodsWrapper,{
                click:true,
                probeType:3   //实时侦测滚动
            });
        },

    2. 在 vue 钩子函数 created 内 this.$nextTick 回调里面调用 better-scroll初始化函数

> 菜单栏根据foodList列表滚动实时高亮

    1. 通过 _calculateHeight 方法动态计算出 每个列表的标题 的 clientHeight 值，并将其推进一个 listHeight 数组
    2. 当滚动 foods 列表时，会动态计算出 pos.y 的值，
    3. 把这个 pos.y 的值在计算属性里判断 其在 listHeight 数组中对应的 index 值
    4. 然后将菜单列表数组中的 index 值 设置为高亮

> 点击左侧菜单栏，右侧 foods 列表实时滚动到相应位置

    1. 给 menu-item 绑定一个 setMenu(index) 方法
    2. 然后根据这个 index 获取foodslist 里面对应的 li dom 元素
    3. 利用 scrollToElement(el,100) api 自动将foodlist滚动到合适位置

    selectMenu(index) {
        // 因为有自动派发事件，所以需要阻止，
        if(!event._constructed) return;
        console.log(index);
        let foodList = this.$refs.foodList;  //通过 $refs.foodList获取当前dom元素
        let el = foodList[index];
        this.foodsScroll.scrollToElement(el,10);
    }

> 购物车计算属性使用

    1. 将 item.foods 数据 通过 props 属性传递到子组件（cartcontrol组件）

    2. 在 cartcontrol 组件内 执行  addCart、decreaseCart 方法改变  item.foods.count 的值

        如果 item.count 值不存在，使用  Vue.set(this.food,'count',1) ;
        给foods增加 count 属性，如果直接增加 count 属性，不会产生响应式数据，必须用  Vue.set() 方法

    3. 在子组件改变 item.foods对象的值，相应的父组件内的 item的值会随之改变（js复杂数据类型地址引用）

    4. 在父组件 goods.vue 利用计算属性 动态的生成购物车数据，然后通过 props属性传递给 shopcart.vue 组件

        计算属性的计算出的值为响应式数据可以直接拿来使用,即在  v-for 中直接遍历  selectFoods

        // 选中的商品即购物车内的商品
        selectFoods() {
            let foods = [];
            this.goods.forEach((good) => {
                good.foods.forEach((food) => {
                    if(food.count){
                        foods.push(food);
                    }
                })
            });
            console.log(foods);
            return foods;
        }



> cartcontrol 增加和减少商品小球动画

    1. 减少商品小球动画
        利用 vue transition 组件-过度动画 和 v-show 配合 可以给任何元素和组件添加 entering/leaving过度
        条件渲染 （使用 v-if）
        条件展示 （使用v-show）
        动态组件
        组件根节点
        当插入或删除包含在 transition 组件中的元素时，Vue将做如下处理：
        1.自动嗅探目标元素是否应用了 css 过度或动画，如果是在恰当的时机添加/删除 css 类名
        2.如果过渡组件提供了 JavaScript钩子函数，这些钩子函数将在恰当的时机被调用
        3.如果没有找到钩子并且也没有检测到css动画，DOM操作（插入/删除）在下一帧中立即执行

        过度的 css 类名
        1. v-enter 定义进入过渡的开始状态，在元素插入式时生效，在下一帧移除
        2. v-enter-active 定义进入过渡的结束状态。在元素被插入时生效，在 transition/animation 完成之后移除
        3. v-leave 定义离开过度的开始状态。在离开过渡被触发时生效，在下一帧移除
        4. v-leave-active 定义离开过渡的结束状态，在离开过渡被触发时生效，在下一帧被移除


        html:
        <transition name="move">
            <!-- 父元素用于控制小球 透明度变化 -->
            <div class="decrease" v-show="food.count>0">
                <!-- 子元素用于控制小球旋转变化 -->
                <span class="inner icon-remove_circle_outline"></span>
            </div>
        </transition>


        css：
        <!-- 小球enter之后最终结束时的状态 -->
        .decrease{
            transition:all 0.4s linear;
            transform:translate3d(0,0,0);
            opacity:1;
            .inner{
                transition:all 0.4s linear;
                transform:rotate(0deg);
            }
        }
        <!-- 小球刚刚enter的状态和小球leave-active状态 -->
        &.move-enter,&.move-leave-active{
            transition:all 0.4s linear;
            transform:translate3d(24px,0,0);
            opacity:0;
            .inner{
                transform:rotate(180deg);
            }
        }


```
2. 增加小球动画

    实现过程：

    1、小球最终的落点都是一致的，在左下角购物车按钮处 （transform:translate(0,0,0)）

    2、传递点击的 dom 对象
        在 cartcontrol 组件里点击 + 时， 将点击的 dom 元素，通过通过 $emit 派发给父组件 goods.vue
        this.$emit('add',event.target);
        <div class="cart-wrapper">
            <!-- add自定义事件用于派发当前点击的dom元素，add为子组件方法，addFood为父组件方法 -->
            <cartcontrol :food="food" @add="addFood"></cartcontrol>
        </div>
        // 子组件$emit派发而来的事件
        addFood(target) {
            this._drop(target);  //传递 target
        },
        _drop(target) {
            // 体验优化,异步执行下落动画
            this.$nextTick(() => {
            //调用 shopcar 组件中的 drop 方法，向 shopcar组件 传入当前点击的 dom 对象
                this.$refs.shopcart.drop(target);
            });
        }

    3.在 shopcar 组件里，创建 小球 dom 结构

        <!-- 小球容器 -->
        <div class="ball-container">
            <div v-for="ball in balls">
                <!-- 过度钩子函数 -->
                <transition name="drop" v-on:before-enter="beforeDrop" v-on:enter="dropping" v-on:after-enter="afterDrop">
                    <!--  外层纵向运动，内层横向运动-->
                    <div class="ball" v-show="ball.show">
                        <div class="inner inner-hook"></div>
                    </div>
                </transition>
            </div>
        </div>


    4. 创建 一个小球数组，内置5个对象（5个小球，均有 show 属性，初始值为false）
        以便在多次快速点击时，屏幕出现多个小球
        5个小球的初始位置 均在 左下角 购物车按钮处
        创建一个 dropBalls 数组用于存储 处在下落过程中的小球
        执行下落时 将 父组件传递过来的 dom 对象 当做一个属性 给 ball，方便 在下面的方法中计算 ball 的位置
        data() {
            return {
                // 创建5个小球用于动画
                balls:[{show:false},{show:false},{show:false},{show:false},{show:false}],
                dropBalls:[], // 存储下落小球
            }
    	},
    5.执行 v-on:before-enter="beforeDrop"  过度前钩子函数
        设置 ball 初始位置，计算处 初始位置与目标位置的 差值 x,y ，将小球 transform ：translate（x,y,0）到动画初始位置

    6.执行 v-on:enter="dropping"  过度中钩子函数
        手动触发浏览器重绘，将 ball 通过 transform ：translate（0，0，0） 移动到目标位置

    7. 执行 v-on:after-enter="afterDrop"  过度结束钩子函数
        从存储下落小球的数组里 unshift 当前小球
        并将当前小球 display:none; show:false

    8.样式
    .ball-container{
        //外层 做纵向运动
        .ball{
            position:fixed
            left:32px
            bottom:22px
            z-index:200
            //y 轴 贝塞尔曲线
            transition:all 2s cubic-bezier(0.49, -0.29, 0.75, 0.41)
            //内从做横向运动
            .inner{
                width:16px
                height:16px
                border-radius:50%
                background-color:rgb(0,160,220)
                //x 轴只需要线性缓动
                transition:all 2s linear
            }
        }

```

> 购物车列表的显示隐藏状态

    ```
        按钮控制 fold => fold 控制 => listShow ， listShow => 控制状态显示 (在totalCount>0)
        在 data 选项里，定义一个 fold（折叠，true） 控制购物车的显示隐藏状态
        在 computed 计算属性里，定义一个 listshow 方法，来表示购物车列表的显示隐藏状态

        listShow() {
            if(!this.totalCount){  //假如所选商品为 0 ，return 掉结果，并将 fold 置为初始值
                this.fold = true;
                return false;
            }
            let show = !this.fold; // 否则，取 fold 的反值，靠 fold 的变化来 决定 列表显示与否
            return show;
        }

        在 method 方法里有个 toggleList 方法控制 fold 状态
        toggleList(){
            if(!this.totalCount){
                return;
            }
            this.fold = !this.fold;
        },
    ```


> 详情页组件

```
    将选中的商品 通过 props 传给 子组件
    <food @add="addFood" :food="seeFoodinfo" ref="food"></food>
    food 组件 通过 $emit 将food 组件添加购物车按钮传递给 父组件 以便实现小球动画

    addFood(target){
        console.log(target);
        //当前组件必须在父组件 引入处，bangding @add="xxx",继而执行 父组件的 xxx 方法
        this.$emit('add',target);
    },

    详情页 过渡动画
    <transition name="fade" ></transition>

    &.fly-enter-active, &.fly-leave-active {
        transition: all 0.2s linear
    }
    &.fly-enter, &.fly-leave-active {
        transform: translate3d(100%, 0, 0)
    }

    ```

> ratingselect 组件（评价选择组件）


```
    1. 评价组件
    全部、推荐、吐槽 类似一个 tab 选项卡的栏目
    只看有内容的评价 筛选
    因为整个项目会有两个地方有这个东西，所以将其抽象为 ratingselect 组件

    组件书写：
    上边是一个 tab 选项卡
    1. 定义 三个常量 代表这三种状态
        const Positive = 0;     //推荐
        const Negative = 1;     //吐槽
        const All = 2;          //全部

        <div class="rating-type border-1px">
            <span @click="select(2,$event)" class="block positive" :class="{'active':selectType===2}">{{desc.all}} <span class="count">{{ratings.length}}</span></span>
        </div>
        在点击事件中，将这三个状态，发送给 父组件
        由于这 三个选项 的 选中状态，是由父组件（food.vue）父组件通过 props 传递过来的，所以不可以在子组件中修改

        select(type,event){
            if(!event._constructed){
                return;
            }
            //不可以在子组件内，随意改变父组件传过来的值，通过 $emit 将子组件需要改变的值，发送给父组件，然后父组件在通过 props 传给 子组件，然后 view 就会发生相应的改变
            this.$emit('select',type);
        }


        父组件：
            使用子组件
            <ratingselect
                @select="selectRating"
                @onlyContent="toggleContent"
                :ratings="food.ratings"
                :selectType="selectType"
                :onlyContent="onlyContent"
                :desc="desc"
            ></ratingselect>

            //在 父组件 methods 对象中 用 selectRating 方法接收子组件 emit 过来的值，赋值给 父组件 selectType 然后在通过 props传递给子组件，从而实现改变
            selectRating(type){
                this.selectType = type;
                this.$nextTick(()=> {
                    this.scroll.refresh();
                })
            },

            //只看有内容的 评价 也是同理

```


> #food.vue 组件中的时间转换函数

```
    在 common 目录下创建一个公共工具函数 utils.js ,然后在需要用到的 组件中，进行 import 引入

    utils.js
        export
            function formatDate(fmt){
                ......
            }


    在 food 组件中使用,只需用 import 引入要使用到的 方法 即可
    import { format } from 'common/js/utils'
    在组件中即可直接使用 该方法
```

>　food.vue 里这种列表布局

```
    上下左右的间距，用 padding 撑开
    左边 用 flex 给个固定的尺寸 flex: 0 0 28px
    右侧 用 flex:1 ，右侧剩余空间 自动充满
    然后右侧内容自然流布局，上下 margin 分配
    右侧时间采用绝对定位
    布局：清晰简单明了

    一般情况下：列表中文字垂直居中的布局一般用 上下 padding 撑开，不要直接设置高度，用line-height居中
    文字高度用 line-height 撑开
```

>  商家页面(seller.vue) 商家实景页面

```
    商家实景左右滚动列表图片

    先根据图片尺寸和左右 margin 计算出 list 列表容器的 宽度，然后 用 better-scroll 进行左右滚动

    一般情况下，要在 vue mounted 之后就可以初始化 better-scroll
    但是这时候，图片资源还没有请求到，所以无法得知 图片的 pics 的 length，继而无法得知，列表容器的宽度

    解决办法：
    vue 提供了一个 watch 对象，来用来监测数据的变化
    当 watch 监测到 seller 数据的变化，然后调用 _initPicScroll，初始化 better-scroll
    watch:{
        'seller'(){
            this.$nextTick(()=>{
                this._initPicScroll();
            })
        }
    },
    methods:{
        _initPicScroll() {
            if(this.seller.pics){
                let picWidth = 120;
                let margin = 6;
                let width = this.seller.pics.length * (picWidth + margin) - margin;
                this.$refs.picList.style.width = width + 'px';

                //better-scroll左右滚动
                this.picScroll = new BScroll(this.$refs.picWrapper,{
                    scrollX: true,
                    eventPassthrough: 'vertical'
                })
            }
        }
    }

```

>　 利用localStorage 在本地收藏商家

```
    收藏商家是放在本地缓存 localStorage 里的

    #１.　在　common/js/utils 文件里创建两个公共函数函数 写入 localStorae 和 读取 localStorage
    # 2.  在点击收藏按钮时，调用存储 方法，首次进入页面时，调用 读取方法

    由于 确定收藏与否的 favorite 属性，是在 data 选项上被vue监测的，所以在data 选项上 favorite 是一个立即执行函数

    data:{
        favorite: ( () => {
            // 要读取的对象，key值，默认值
            return loadFromLocal(this.seller.id, 'favorite', false);
        } )()
    }
```
>   路由切换时，各组件会保持原来的状态

```
    # 在路由外连上加上  <keep-alive> 即可
    <!-- 路由外链 -->
        <keep-alive>
            <router-view :seller="seller"></router-view>
        </keep-alive>
```