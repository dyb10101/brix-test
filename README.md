Brix-实际项目
=========

##项目背景

![项目背景](http://img01.taobaocdn.com/tps/i1/T1DWzOXmxdXXXUBqn8-3663-1304.png)


##配置初始化

    <link type="text/css" rel="stylesheet" href="http://a.tbcdn.cn/apps/e/brix/1.0/brix.css" charset="utf-8">
    <script src="http://a.tbcdn.cn/s/kissy/1.2.0/kissy.js"></script>
    <!-- 自动读取节点配置 -->
    <script src="http://a.tbcdn.cn/apps/e/brix/1.0/brix.js" bx-config="{componentsPath:'./',importsPath:'./'}"></script>


##快速实现组件调用

以后一个页面的brix组件会有三个来源

* brix/gallery/name/ ：核心组件
* components/name/: 项目组件
* inports/namespace/name/ ：其他项目成熟的组件

这三块如何整合呢？
其实很简单，页面所有的组件，无论其来源是哪里，其实只要是遵循brix规范写出来的，都是一样的用法。

* 定义模板、配置hooks

gallery模板一

    <div>
        <ul id="kwicks1" class="kwicks" bx-name="kwicks" bx-config="{max:205,spacing:5,autoplay:true}">
            {{#brixkwick}}
            <li class="kwick{{.}}"></li>
            {{/brixkwick}}
        </ul>
    </div>

components模板二
    
    <div>
        <ul id="ulkwicks1" class="kwicks" bx-name="kwicks" bx-path="components/kwicks/" bx-config="{max:205,spacing:5,autoplay:true}">
            <li class="kwick1"></li>
            <li class="kwick2"></li>
            <li class="kwick3"></li>
            <li class="kwick4"></li>
        </ul>
    </div>
    

imports模板三

    <div>
        <ul class="kwicks" bx-name="kwicks" bx-path="imports/etao.ux.x2/kwicks/" bx-config="{max:205,spacing:5,autoplay:true}">
            <li class="kwick1"></li>
            <li class="kwick2"></li>
            <li class="kwick3"></li>
            <li class="kwick4"></li>
        </ul>
    </div>

* 定义数据

模板需要的所有数据

    var pagelet_data = {
        brixkwick: [1, 2, 3, 4]
    };

* new Pagelet
    
一个pagelet搞定所有

    KISSY.use('brix/core/pagelet', function(S, Pagelet) {
        var pagelet = new Pagelet({
            container:'#container',//容器 //如果tmpl自定的节点（非script节点）已经在dom中，则不需要容器
            tmpl:'#tmpl_script',//模板来源，也可以是页面已经存在的dom节点
            data:pagelet_data,//数据
            autoRender:true， //是否自动渲染，如果false，则需手工调用render方法
            behavior:true //是否自动构建组件，如果false，则需手工调用addBehavior方法
        })；
        //pagelet.render();
        pagelet.ready(function(){
            //完成渲染和行为附加
            var kwicks1 = pagelet.getBrcik('kwicks1');
            //kwicks1：就和你平常new某个组件的实例对象。
        });
        //pagelet1.addBehavior();
    });


##快速搭建demo页面

组件写完会有默认的模板和数据，和index.js放在同一级目录下，命名template.html和data.json

###配置布局模板和“坑”

    <h2>项目components下的组件</h2>
    <h4>ext1</h4>
    <div id="example1">
        {{@components/kwicks/ext1/}}
    </div>
    <a href="#" class="btn">销毁</a>
    <h4>ext2</h4>
    <div id="example7">
        {{@components/kwicks/ext2/}}
    </div>
    <h2>项目imports下的组件</h2>
    <h4>ext1</h4>
    {{@imports/etao.ux.x2/breadcrumbs/ext1/}}


####布局

就是普通的完整html代码，可以穿插mustache模板，组件用特殊的占位符号（坑）

####坑

{{@path}}:{{@components/kwicks/}}

* path:完整的组件路径。
* “@”：可以自行决定用什么，如果和后端语言或则其他模板语言冲突，可以修改


###xhr同步读取template.html和data.json


    KISSY.use('brix/core/pagelet', function(S, Pagelet) {
                var tmpl = S.one('#tmpl_script').html();
                var pagelet_data = {};
                var s = '@';
                reg = new RegExp('\{\{'+s+'(.+)?\}\}',"ig");
                tmpl = tmpl.replace(reg,function($1,$2){
                    S.log($2);
                    var str = '';
                    var p = $2.replace(/\//ig,'_').replace(/\./ig,'_');
                    pagelet_data[p] = pagelet_data[p] || {};
                    S.io({
                        url:$2+'template.html',
                        async:false,
                        success:function(data , textStatus , xhrObj){
                            str = '{{#'+p+'}}' + data+'{{/'+p+'}}';
                        }
                    });
                    S.io({
                        url:$2+'data.json',
                        async:false,
                        dataType:'json',
                        success:function(data , textStatus , xhrObj){
                            for(var k in data){
                                pagelet_data[p][k] = data[k];
                            }
                        }
                    });
                    return str;
                });

                var pagelet = new Pagelet({
                    container:'body',
                    tmpl:tmpl,
                    data:pagelet_data,
                    autoRender:true,
                    callback:function(){
                        S.one('.btn').on('click',function(){
                            pagelet.getBrick(S.all('.kwicks').item(0).attr('id')).destroy();
                        });
                    }
                });
                S.log(tmpl);
            });







