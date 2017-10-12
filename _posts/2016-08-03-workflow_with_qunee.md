---
layout: post
title: 工作流程图的树形展示
category: programming
---

实际工作中遇到一个问题，在html页面以树形节点的方式展示工作流程图。
echarts2的树形图可以实现，但是页面样式不够灵活。架构师从网上找来qunee插件，大致能满足客户要求。
qunee是"一套基于HTML5的网络图组件"，详情可以去官网http://qunee.com 查看。
这里只介绍如何用qunee来实现树形图。
在qunee的demo里找到
Treelayouter Demo (<a href="http://demo.qunee.com/#TreeLayouter Demo" target="_blank">```http://demo.qunee.com/#TreeLayouter Demo```</a>)
和Work Flow Demo中的
development guide (<a href="http://demo.qunee.com/#TreeLayouter Demo" target="_blank">```http://demo.qunee.com/#Development Guide```</a>)
本例需要将两者的效果结合。

流程图结构如下(局部)：
![](http://upload-images.jianshu.io/upload_images/1817197-08ea5e34baf34dc2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
流程图结构如下(完整)：
![](http://upload-images.jianshu.io/upload_images/1817197-a4b2b936e5ebbc7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
节点数据是json格式，举例如下：

	var test = {  
    "children": [{  
        "children": [{  
            "children": null,  
            "codeDescription": "2级节点",  
            "nodeLevel": 2,  
            "nodeName": "test1_1",  
            "nodeOrder": 0  
        }],  
        "codeDescription": "1级节点",  
        "nodeLevel": 1,  
        "nodeName": "test1",  
        "nodeOrder": 0  
    }],  
    "codeDescription": "根节点",  
    "nodeLevel": 0,  
    "nodeName": "test0",  
    "nodeOrder": 0  
	}

完整的html代码，每一部分都有较详细的注释。

	<!DOCTYPE html>  
	<html>  
	<head>  
	<meta charset="utf-8" />  
	<meta http-equiv="X-UA-Compatible" content="IE=edge" />  
	<title>Qunee Treelayout Demo 20160801 update</title>  
	<!--引入qunee插件的js文件-->  
	<script src="./qunee-min.js"></script>  
	<!--引入json数据 -->  
	<script src="./dataserver1.js"></script>  
	
	
	</head>  
	<body>  
	    <div id="root_box"  
	        style="width: 1000px; height: 600px; margin: auto; border: solid 1px #2898E0;"></div>  
	
	    <!-- <script type="text/javascript" src="work-process.js"></script> -->  
	    <script type="text/javascript">  
	        //指定一个div元素，初始化qunee画布  
	        var graph = new Q.Graph("root_box");  
	        //graph.originAtCenter为false时表示设置左上角为坐标原点  
	        graph.originAtCenter = false;  
	
	        //创建一个数组存放所有的子孙节点  
	        var allChildren = [];  
	
	        //用来记录最大层级  
	        var maxLevel = 1;  
	
	        //递归函数，将传入的数据创建成树形图的节点  
	        function loadDatas(json, parent, layoutType) {  
	            json.forEach(function(data) {  
	                //获取最深的层级  
	                if (data.nodeLevel > maxLevel) {  
	                    maxLevel = data.nodeLevel;  
	                }  
	
	                var node = createNode(data.nodeName, parent, layoutType);  
	                allChildren.push(node);  
	
	                if (layoutType == Q.Consts.LAYOUT_TYPE_EVEN_VERTICAL) {  
	                    node.vGap = 20; //设置孩子布局的垂直间距,hGap为水平间距  
	                }  
	
	                //递归创建所有节点  
	                if (data.children) {  
	                    loadDatas(data.children, node, Q.Consts.LAYOUT_TYPE_EVEN_VERTICAL);  
	                }  
	            });  
	        }  
	
	        //传入根节点(根节点属性中包含了所有子孙节点),生成树形图  
	        function init(rootNode) {  
	            //建立一个数组用于存放主流程节点  
	            var nodeL1Arr = [];  
	
	            //在本demo中，rootNode根节点不展示，根节点下的第一级节点作为主流程节点  
	            if (rootNode.children) {  
	                rootNode.children.forEach(function(nodeL1) {  
	                    var newNode = createStep(nodeL1.nodeName);  
	                    nodeL1Arr.push(newNode);  
	                    if (nodeL1.children) {  
	                        loadDatas(nodeL1.children, newNode, Q.Consts.LAYOUT_TYPE_EVEN_VERTICAL);  
	                    }  
	                });  
	            } else {  
	                var newNode = createNode("没有流程信息");  
	            }  
	            //设置树形图布局  
	            setLayout(nodeL1Arr);  
	        }  
	
	        //创建单个流程节点  
	        function createNode(name, from, layoutType) {  
	            var node = graph.createText(name);  
	            node.setStyle(Q.Styles.LABEL_BORDER, 1);  
	            node.setStyle(Q.Styles.LABEL_BORDER_STYLE, "#1D4876");  
	            node.setStyle(Q.Styles.LABEL_FONT_SIZE, 16);  
	            node.setStyle(Q.Styles.LABEL_PADDING, 5);  
	            node.setStyle(Q.Styles.LABEL_SIZE, new Q.Size(70, 35));  
	            node.setStyle(Q.Styles.LABEL_BACKGROUND_COLOR, "#FFF");  
	            //节点是否可见  
	            node.visible = true;  
	            //节点是否能用鼠标拖动,false为不能拖动  
	            node.movable = false  
	            node.layoutType = layoutType;  
	
	            if (from) {  
	                node.parent = from;  
	                node.host = from;  
	            }  
	
	            if (from instanceof Q.Node) {  
	                //创建连线  
	                var nodeEdge = graph.createEdge(from, node);  
	                if (from.layoutType == Q.Consts.LAYOUT_TYPE_EVEN_VERTICAL) {  
	                    nodeEdge.edgeType = Q.Consts.EDGE_TYPE_VERTICAL_HORIZONTAL;  
	                } else {  
	                    nodeEdge.edgeType = Q.Consts.EDGE_TYPE_ORTHOGONAL;  
	                }  
	            }  
	            return node;  
	        }  
	
	        //创建主流程节点  
	        function createStep(label) {  
	            var node = graph.createText(label);  
	            node.setStyle(Q.Styles.LABEL_BORDER, 1);  
	            node.setStyle(Q.Styles.LABEL_BACKGROUND_COLOR, "#FFF");  
	            node.setStyle(Q.Styles.LABEL_BORDER_STYLE, "#1D4876");  
	            node.setStyle(Q.Styles.LABEL_FONT_SIZE, 20);  
	            node.setStyle(Q.Styles.LABEL_SIZE, new Q.Size(120, 50));  
	            node.visible = true;  
	            //节点是否能用鼠标拖动,false为不能拖动  
	            node.movable = false  
	            node.layoutType = Q.Consts.LAYOUT_TYPE_EVEN_VERTICAL;  
	            //node.vGap = 30;//设置孩子布局的垂直间距,hGap为水平间距  
	
	            return node;  
	        }  
	
	        //创建连线  
	        function createEdge(from, to, lineWidth, dash) {  
	            var edge = graph.createEdge(from, to);  
	            edge.setStyle(Q.Styles.EDGE_WIDTH, lineWidth || 3);  
	            edge.setStyle(Q.Styles.EDGE_COLOR, "#1D4876");  
	            if (dash) {  
	                edge.setStyle(Q.Styles.EDGE_LINE_DASH, [ 10, 10 ]);  
	            }  
	            return edge;  
	        }  
	
	        //调整树状图各分支的位置，形成美观的对称结构  
	        function moveNodes() {  
	            var rightBound = 0; //节点右边界  
	            var rightX = 0 //用于记录最右侧节点的横坐标  
	            var rightElementName; //边界节点的名称  
	            var prevElement; //前一主流程  
	            var dx = 20 * maxLevel; //移动基数20乘以获取的最大层级  
	            graph.graphModel.forEachByTopoBreadthFirstSearch(function(element) {  
	                if (element instanceof Q.Node) {  
	                    //每次移动，整个父子节点链的线条都会移动  
	                    graph.moveElements([ element ], dx, 0);  
	
	                    if (!element.parent) {  
	                        //如果节点横坐标小于上一节点的右边界，则移动节点，避免重合  
	                        if (element.x < rightBound) {  
	                            graph.moveElements([ element ], rightBound - element.x, 0);  
	                        }  
	
	                        //如果主流程没有子流程，需要单独移动固定距离dx，否则连线会非常短  
	                        if (prevElement && !prevElement.hasChildren()) {  
	                            graph.moveElements([ element ], dx, 0);  
	                        }  
	                        prevElement = element;  
	                    }  
	
	                    if (element.x >= rightX) {  
	                        //更新最右侧节点横坐标的值  
	                        rightX = element.x;  
	
	                        //rightX为最右侧节点的横坐标，加上该节点边框的宽度，得到最右侧边界的位置  
	                        rightBound = rightX + graph.getUIBounds(element).width;  
	
	                        //获取最右侧节点的名称，用于打印测试  
	                        rightElementName = element.name;  
	                    }  
	                }  
	            });  
	        }  
	
	        graph.visibleFilter = function(node) {  
	            return node.visible !== false;  
	        }  
	
	        var nodeClicked;  
	        // 设置点击事件  
	        graph.onclick = function(evt) {  
	            nodeClicked = evt.getData();  
	
	            if (!nodeClicked) {  
	                Q.forEach(allChildren, function(p) {  
	                    p.visible = true;  
	                    p.invalidateVisibility();  
	                })  
	                graph.invalidate();  
	                return;  
	            }  
	
	            //点击主流程节点，只显示子孙节点，隐藏其他节点  
	            /* if (!nodeClicked.parent && !nodeClicked.from) {  
	                Q.log(nodeClicked);  
	                Q.forEach(allChildren, function(p) {  
	                    var visible = p.isDescendantOf(nodeClicked);  
	                    p.visible = visible;  
	                    p.invalidateVisibility();  
	                })  
	                graph.invalidate();  
	            } */  
	
	            //点击主流程节点，显示或隐藏其子孙节点  
	            if (!nodeClicked.parent && !nodeClicked.from&&nodeClicked.hasChildren()) {  
	                setAllChildren(nodeClicked);  
	                graph.invalidate();  
	            }   
	        }  
	
	        //显示或隐藏子孙节点的函数  
	        function setAllChildren(parent){  
	            Q.forEach(parent.children, function(p) {  
	                p.visible = !p.visible;  
	                p.invalidateVisibility();  
	                if(p.hasChildren()){  
	                    setAllChildren(p);  
	                }  
	            })  
	        }  
	
	        //设置树形图布局  
	        var layouter = new Q.TreeLayouter(graph);  
	        function setLayout(nodeL1Arr) {  
	            layouter.layoutType = Q.Consts.LAYOUT_TYPE_EVEN_HORIZONTAL;  
	            //qunee新特性，节点连线能够等长排列  
	            layouter.parentChildrenDirection = Q.Consts.DIRECTION_BOTTOM_RIGHT;  
	            layouter.doLayout({  
	                callback : function() {  
	                    //graph.moveToCenter(1);  
	                    graph.zoomToOverview();  
	                    moveNodes();  
	                    if (nodeL1Arr.length > 0) {  
	                        var index = 0  
	                        nodeL1Arr.forEach(function(obj) {  
	                            if (index > 0) {  
	                                createEdge(nodeL1Arr[index - 1], obj);  
	                            }  
	                            index++;  
	                        });  
	                    }  
	                }  
	            });  
	        }  
	        init(test);  
	    </script>  
	</body>  
	</html>

完整demo的下载地址：
[http://download.csdn.net/detail/bigablecat/9593507](http://download.csdn.net/detail/bigablecat/9593507)