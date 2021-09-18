<template>
  <div class="logic-flow-view">
    <!-- <h3 class="demo-title">LogicFlow Turbo Adpter</h3> -->
    <!-- 辅助工具栏 -->
    <Control 
      class="demo-control"
      v-if="lf"
      :lf="lf"
      :catTurboData= 'true'
      @catData="$_catData"
      @catTurboData="$_catTurboData"
    ></Control>
    <!-- 节点面板 -->
    <NodePanel :lf="lf" :nodeList="nodeList"></NodePanel>
    <!-- 画布 -->
    <div id="LF-Turbo"></div>
    <!-- 数据查看面板 -->
    <el-dialog
      title="数据"
      :visible.sync="dataVisible"
      width="50%">
      <DataDialog :graphData="graphData"></DataDialog>
    </el-dialog>
    <!-- <h4>更多示例：
      <el-button type="text" @click="goto">LogicFlow</el-button>
    </h4> -->
    <div v-if="nodeInfo" :key="nodeKey">节点属性{{$data.clickNode}}</div>
   
  </div>
</template>
<script>
import LogicFlow from '@logicflow/core'
// BpmnXmlAdapter
import { Snapshot, BpmnElement } from '@logicflow/extension'
import '@logicflow/core/dist/style/index.css'
import '@logicflow/extension/lib/style/index.css'
import NodePanel from './LFComponents/NodePanel'
import Control from './LFComponents/Control'
import DataDialog from './LFComponents/DataDialog'
// toLogicflowData
import { toTurboData,toLogicflowData } from '../Util/AdpterForTurbo';
import { BpmnNode } from './config'
const demoData = require('./dataTurbo.json')
// const demoData = `<bpmn:definitions id="Definitions_1t5t078" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:bpmn="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:dc="http://www.omg.org/spec/DD/20100524/DC" xmlns:di="http://www.omg.org/spec/DD/20100524/DI" targetNamespace="http://bpmn.io/schema/bpmn" exporter="bpmn-js (https://demo.bpmn.io)" exporterVersion="7.3.0"> <bpmn:process id="Process_2u3iktp" isExecutable="false"> <bpmn:startEvent id="Event_2715eoc" name="开始"> <bpmn:outgoing>Flow_33j4jra</bpmn:outgoing></bpmn:startEvent> <bpmn:userTask id="Activity_06ejpb9" name="测试1"> <bpmn:incoming>Flow_33j4jra</bpmn:incoming> <bpmn:outgoing>Flow_0std6pu</bpmn:outgoing></bpmn:userTask> <bpmn:userTask id="Activity_2isko2b" name="测试2"> <bpmn:incoming>Flow_0std6pu</bpmn:incoming> <bpmn:outgoing>Flow_081nvcj</bpmn:outgoing></bpmn:userTask> <bpmn:endEvent id="Event_3o35ddb" name="结束"> <bpmn:incoming>Flow_081nvcj</bpmn:incoming></bpmn:endEvent> <bpmn:sequenceFlow id="Flow_33j4jra" sourceRef="Event_2715eoc" targetRef="Activity_06ejpb9"/> <bpmn:sequenceFlow id="Flow_0std6pu" sourceRef="Activity_06ejpb9" targetRef="Activity_2isko2b"/> <bpmn:sequenceFlow id="Flow_081nvcj" sourceRef="Activity_2isko2b" targetRef="Event_3o35ddb"/> </bpmn:process> <bpmndi:BPMNDiagram id="BPMNDiagram_1"> <bpmndi:BPMNPlane id="BPMNPlane_1" bpmnElement="Process_2u3iktp"> <bpmndi:BPMNEdge id="Flow_33j4jra_di" bpmnElement="Flow_33j4jra"> <di:waypoint x="318" y="220"/> <di:waypoint x="374" y="220"/> <di:waypoint x="374" y="240"/> <di:waypoint x="430" y="240"/> </bpmndi:BPMNEdge> <bpmndi:BPMNEdge id="Flow_0std6pu_di" bpmnElement="Flow_0std6pu"> <di:waypoint x="530" y="240"/> <di:waypoint x="650" y="240"/> </bpmndi:BPMNEdge> <bpmndi:BPMNEdge id="Flow_081nvcj_di" bpmnElement="Flow_081nvcj"> <di:waypoint x="750" y="240"/> <di:waypoint x="842" y="240"/> </bpmndi:BPMNEdge> <bpmndi:BPMNShape id="Event_2715eoc_di" bpmnElement="Event_2715eoc"> <dc:Bounds x="280" y="200" width="40" height="40"/> <bpmndi:BPMNLabel> <dc:Bounds x="290" y="213" width="20" height="14"/></bpmndi:BPMNLabel></bpmndi:BPMNShape> <bpmndi:BPMNShape id="Activity_06ejpb9_di" bpmnElement="Activity_06ejpb9"> <dc:Bounds x="430" y="200" width="100" height="80"/> <bpmndi:BPMNLabel> <dc:Bounds x="465" y="233" width="30" height="14"/></bpmndi:BPMNLabel></bpmndi:BPMNShape> <bpmndi:BPMNShape id="Activity_2isko2b_di" bpmnElement="Activity_2isko2b"> <dc:Bounds x="650" y="200" width="100" height="80"/> <bpmndi:BPMNLabel> <dc:Bounds x="685" y="233" width="30" height="14"/></bpmndi:BPMNLabel></bpmndi:BPMNShape> <bpmndi:BPMNShape id="Event_3o35ddb_di" bpmnElement="Event_3o35ddb"> <dc:Bounds x="840" y="220" width="40" height="40"/> <bpmndi:BPMNLabel> <dc:Bounds x="850" y="233" width="20" height="14"/></bpmndi:BPMNLabel></bpmndi:BPMNShape> </bpmndi:BPMNPlane></bpmndi:BPMNDiagram></bpmn:definitions>`

export default {
  name: 'LF',
   components: { NodePanel, Control, DataDialog },
  data () {
    return {
      lf: null,
      dialogVisible: false,
      graphData: null,
      dataVisible: false,
      // config: {
      //   grid: true,
      //   background: {
      //     color: '#f7f9ff'
      //   },
      //   keyboard: {
      //     enabled: true
      //   },
      // },
          config: {
        background: {
          color: '#f7f9ff'
        },
        grid: {
          size: 10,
          visible: false
        },
        keyboard: {
          enabled: true
        },
        style: {
          rect: {
            radius: 6,
          },
          edgeText: { // 边文本样式
            background: {
              fill: '#fff'
            }
          },
        },
        edgeTextDraggable: true,
        guards: {
          beforeClone (data) {
            console.log('beforeClone', data)
            return true
          },
          beforeDelete (data) {
            // 可以根据data数据判断是否允许删除，允许返回true,不允许返回false
            // 文档： http://logic-flow.org/guide/basic/keyboard.html#%E5%A6%82%E4%BD%95%E9%98%BB%E6%AD%A2%E5%88%A0%E9%99%A4%E6%88%96%E8%80%85%E6%8B%B7%E8%B4%9D%E8%A1%8C%E4%B8%BA
            console.log('beforeDelete1111', data)
            // _this.$message('不允许删除', 'error')
            return true
          }
        }
      },
      nodeList: BpmnNode,
      nodeInfo:false,
      nodeKey:0
    }
  },
  mounted () {
    this.$_initLf()
  },
  methods: {
    $_initLf () {
      // 画布配置
      LogicFlow.use(Snapshot)
      // 使用bpmn插件，引入bpmn元素，这些元素可以在turbo中转换后使用
      LogicFlow.use(BpmnElement)
      const lf = new LogicFlow({...this.config, container: document.querySelector('#LF-Turbo')})
      this.lf = lf
        // 设置主题
      lf.setTheme({
        circle: {
          r: 20,
          stroke: '#000000',
          outlineColor: '#88f',
          strokeWidth: 1
        },
        rect: {
          outlineColor: '#88f',
          strokeWidth: 1
        },
        polygon: {
          strokeWidth: 1
        },
        polyline: {
          stroke: '#000000',
          hoverStroke: '#000000',
          selectedStroke: '#000000',
          outlineColor: '#88f',
          strokeWidth: 1
        },
        nodeText: {
          color: '#000000'
        },
        edgeText: {
          color: '#000000',
          background: {
            fill: '#f7f9ff'
          }
        }
      })
      // 设置边类型bpmn:sequenceFlow为默认类型
      lf.setDefaultEdgeType('bpmn:sequenceFlow')
      this.$_render()
    },
       $_LfEvent () {
      this.lf.on('node:click', ({data}) => {
        console.log('node:click', data)

    data.properties = {ceshi:'这里是属性',}

      this.lf.setNodeData(data);
        this.$data.clickNode = data
        this.nodeInfo = true
        this.nodeKey += 1
        
        // this.$data.dialogVisible = true
      })
      this.lf.on('edge:click', ({data}) => {
        console.log('edge:click', data)
        data.properties = {ceshi:'这里是属性'}
      this.lf.setEdgeData(data);

         this.$data.clickNode = data
               this.nodeInfo = true
        this.nodeKey += 1
          // this.$data.dialogVisible = true
      })
          this.lf.on('blank:click', () => {
        this.nodeInfo = false
      })
  
    },
    $_render () {
      // Turbo数据转换为LogicFlow内部识别的数据结构
      const lFData = toLogicflowData(demoData)
      this.lf.render(lFData)
      // this.lf.render(demoData)
       this.$_LfEvent()
    },
    closeDialog () {
      this.$data.dialogVisible = false
    },
    $_catData(){
      this.$data.graphData = this.$data.lf.getGraphData();
      this.$data.dataVisible = true;
    },
    $_catTurboData(){
      let graphData = this.$data.lf.getGraphData();
      graphData.nodes = graphData.nodes.map((q)=>{
        let outgoingArr = []
        graphData.edges.forEach((j)=>{
 if(q.id === j.sourceNodeId) {
   outgoingArr.push(j.targetNodeId)
 }

        })
        q.outgoing = outgoingArr 
 return q
      })
      console.log(graphData,'graphData--')
      // 数据转化为Turbo识别的数据结构
      this.$data.graphData = toTurboData(graphData)
    //  const jsonData = toTurboData(graphData)

      this.$data.dataVisible = true;
    },
    goto () {
      this.$router.push('/')
    }
  }
}
</script>
<style>
.logic-flow-view {
  height: 100vh;
  position: relative;
}
.demo-title{
  text-align: center;
  margin: 20px;
}
.demo-control{
  position: absolute;
  top: 50px;
  right: 50px;
  z-index: 2;
}
#LF-Turbo{
  width: calc(100% - 100px);
  height: 80%;
  outline: none;
  margin-left: 50px;
}
.time-plus{
  cursor: pointer;
}
.add-panel {
  position: absolute;
  z-index: 11;
  background-color: white;
  padding: 10px 5px;
}
.el-drawer__body {
  height: 80%;
  overflow: auto;
  margin-top: -30px;
  z-index: 3;
}
</style>

