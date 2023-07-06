<script setup>
import { VueFlow, useVueFlow, Position, MarkerType  } from '@vue-flow/core'
import { nextTick, watch,  } from 'vue'
import { Background, Controls,  MiniMap } from '@vue-flow/additional-components'

import './styles/main.css'
import ToolbarNode from './components/ToolbarNode.vue'
import PanelButton  from './components/PanelButton.vue'
import SideBar from './components/Side-Bar.vue'
import ControlJobs from './components/ControlsJobs.vue'


let id = 0
function getId() {
  return `dndnode_${id++}`
}

const defaultNodeStyle = {
  border: '1px solid #10b981',
  background: '#9400D3',
  color: 'white',
  borderRadius: '99px',
}

const { findNode, onConnect, addEdges, addNodes, project, vueFlowRef} = useVueFlow({
  // nodes: [
  //     {
  //         id: '1',
  //         type: 'smoothstep',
  //         label: 'teste',
  //         data: { toolbarPosition: Position.Top },
  //         position: { x: 200, y: 0 },
  //         class: 'light',
  //         animated: true
  //         //style: defaultNodeStyle,
        
  //     },
  //   ],
})

function onDragOver(event) {
  event.preventDefault()

  if (event.dataTransfer) {
    event.dataTransfer.dropEffect = 'move'
  }
}

onConnect((params) => addEdges(params))


function onDrop(event) {
  const type = event.dataTransfer?.getData('application/vueflow')
  console.log(type)
  const { left, top } = vueFlowRef.value.getBoundingClientRect()

  const position = project({
    x: event.clientX - left,
    y: event.clientY - top,
  })

  const newNode = {
    id: getId(),
    type: 'toolbar',
    position,    
    label: `${type} node`,       
    data: { toolbarPosition: Position.Top },    
    style: defaultNodeStyle,
       
  }

  addNodes([newNode])
  
  // align node position after drop, so it's centered to the mouse
  nextTick(() => {
    const node = findNode(newNode.id)
    const stop = watch(
      () => node.dimensions,
      (dimensions) => {
        if (dimensions.width > 0 && dimensions.height > 0) {
          node.position = { x: node.position.x - node.dimensions.width / 2, y: node.position.y - node.dimensions.height / 2 }
          stop()
        }
      },
      { deep: true, flush: 'post' },
    )
  })
}

</script>

<template>
  <div class="dndflow" @drop="onDrop">
    <VueFlow @dragover="onDragOver" 
             class="basicflow" 
             :class="{ dark }" 
             :default-edge-options="{ type: 'smoothstep',  markerEnd: MarkerType.Arrow }"
             fit-view-on-init>
     
      <template #node-toolbar="nodeProps">
        <ToolbarNode :data="nodeProps.data" :label="nodeProps.label" />
      </template> 

      <MiniMap />
      <Controls />
      <ControlJobs />
      
      <PanelButton /> 
      
      <Background :pattern-color="dark ? '#FFFFFB' : '#aaa'" gap="8" /> 
    </VueFlow>

    <SideBar />
    
  </div>  
</template>
