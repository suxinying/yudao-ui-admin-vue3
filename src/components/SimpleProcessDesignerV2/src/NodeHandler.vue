<template>
  <div class="node-handler-wrapper">
    <div class="node-handler">
      <el-popover
        trigger="hover"
        v-model:visible="popoverShow"
        placement="right-start"
        width="auto"
        v-if="!readonly"
      >
        <div class="handler-item-wrapper">
          <div class="handler-item" @click="addNode(NodeType.USER_TASK_NODE)">
            <div class="approve handler-item-icon">
              <span class="iconfont icon-approve icon-size"></span>
            </div>
            <div class="handler-item-text">审批人</div>
          </div>
          <div class="handler-item" @click="addNode(NodeType.COPY_TASK_NODE)">
            <div class="handler-item-icon copy">
              <span class="iconfont icon-size icon-copy"></span>
            </div>
            <div class="handler-item-text">抄送</div>
          </div>
          <div class="handler-item" @click="addNode(NodeType.CONDITION_BRANCH_NODE)">
            <div class="handler-item-icon condition">
              <span class="iconfont icon-size icon-exclusive"></span>
            </div>
            <div class="handler-item-text">条件分支</div>
          </div>
          <div class="handler-item" @click="addNode(NodeType.PARALLEL_BRANCH_NODE)">
            <div class="handler-item-icon parallel">
              <span class="iconfont icon-size icon-parallel"></span>
            </div>
            <div class="handler-item-text">并行分支</div>
          </div>
          <div class="handler-item" @click="addNode(NodeType.INCLUSIVE_BRANCH_NODE)">
            <div class="handler-item-icon inclusive">
              <span class="iconfont icon-size icon-inclusive"></span>
            </div>
            <div class="handler-item-text">包容分支</div>
          </div>
          <div class="handler-item" @click="addNode(NodeType.DELAY_TIMER_NODE)">
            <!-- TODO @芋艿 需要更换一下iconfont的图标 -->
            <div class="handler-item-icon copy">
              <span class="iconfont icon-size icon-copy"></span>
            </div>
            <div class="handler-item-text">延迟器</div>
          </div>
        </div>
        <template #reference>
          <div class="add-icon"><Icon icon="ep:plus" /></div>
        </template>
      </el-popover>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  ApproveMethodType,
  AssignEmptyHandlerType,
  AssignStartUserHandlerType,
  NODE_DEFAULT_NAME,
  NodeType,
  RejectHandlerType,
  SimpleFlowNode
} from './consts'
import { generateUUID } from '@/utils'

defineOptions({
  name: 'NodeHandler'
})

const message = useMessage() // 消息弹窗

const popoverShow = ref(false)
const props = defineProps({
  childNode: {
    type: Object as () => SimpleFlowNode,
    default: null
  },
  currentNode: {
    type: Object as () => SimpleFlowNode,
    required: true
  }
})
const emits = defineEmits(['update:childNode'])

const readonly = inject<Boolean>('readonly') // 是否只读

const addNode = (type: number) => {
  // 校验：条件分支、包容分支后面，不允许直接添加并行分支
  if (
    type === NodeType.PARALLEL_BRANCH_NODE &&
    [NodeType.CONDITION_BRANCH_NODE, NodeType.INCLUSIVE_BRANCH_NODE].includes(
      props.currentNode?.type
    )
  ) {
    message.error('条件分支、包容分支后面，不允许直接添加并行分支')
    return
  }

  popoverShow.value = false
  if (type === NodeType.USER_TASK_NODE) {
    const id = 'Activity_' + generateUUID()
    const data: SimpleFlowNode = {
      id: id,
      name: NODE_DEFAULT_NAME.get(NodeType.USER_TASK_NODE) as string,
      showText: '',
      type: NodeType.USER_TASK_NODE,
      approveMethod: ApproveMethodType.SEQUENTIAL_APPROVE,
      // 超时处理
      rejectHandler: {
        type: RejectHandlerType.FINISH_PROCESS
      },
      timeoutHandler: {
        enable: false
      },
      assignEmptyHandler: {
        type: AssignEmptyHandlerType.APPROVE
      },
      assignStartUserHandlerType: AssignStartUserHandlerType.START_USER_AUDIT,
      childNode: props.childNode
    }
    emits('update:childNode', data)
  }
  if (type === NodeType.COPY_TASK_NODE) {
    const data: SimpleFlowNode = {
      id: 'Activity_' + generateUUID(),
      name: NODE_DEFAULT_NAME.get(NodeType.COPY_TASK_NODE) as string,
      showText: '',
      type: NodeType.COPY_TASK_NODE,
      childNode: props.childNode
    }
    emits('update:childNode', data)
  }
  if (type === NodeType.CONDITION_BRANCH_NODE) {
    const data: SimpleFlowNode = {
      name: '条件分支',
      type: NodeType.CONDITION_BRANCH_NODE,
      id: 'GateWay_' + generateUUID(),
      childNode: props.childNode,
      conditionNodes: [
        {
          id: 'Flow_' + generateUUID(),
          name: '条件1',
          showText: '',
          type: NodeType.CONDITION_NODE,
          childNode: undefined,
          conditionType: 1,
          defaultFlow: false
        },
        {
          id: 'Flow_' + generateUUID(),
          name: '其它情况',
          showText: '未满足其它条件时，将进入此分支',
          type: NodeType.CONDITION_NODE,
          childNode: undefined,
          conditionType: undefined,
          defaultFlow: true
        }
      ]
    }
    emits('update:childNode', data)
  }
  if (type === NodeType.PARALLEL_BRANCH_NODE) {
    const data: SimpleFlowNode = {
      name: '并行分支',
      type: NodeType.PARALLEL_BRANCH_NODE,
      id: 'GateWay_' + generateUUID(),
      childNode: props.childNode,
      conditionNodes: [
        {
          id: 'Flow_' + generateUUID(),
          name: '并行1',
          showText: '无需配置条件同时执行',
          type: NodeType.CONDITION_NODE,
          childNode: undefined
        },
        {
          id: 'Flow_' + generateUUID(),
          name: '并行2',
          showText: '无需配置条件同时执行',
          type: NodeType.CONDITION_NODE,
          childNode: undefined
        }
      ]
    }
    emits('update:childNode', data)
  }
  if (type === NodeType.INCLUSIVE_BRANCH_NODE) {
    const data: SimpleFlowNode = {
      name: '包容分支',
      type: NodeType.INCLUSIVE_BRANCH_NODE,
      id: 'GateWay_' + generateUUID(),
      childNode: props.childNode,
      conditionNodes: [
        {
          id: 'Flow_' + generateUUID(),
          name: '包容条件1',
          showText: '',
          type: NodeType.CONDITION_NODE,
          childNode: undefined,
          defaultFlow: false
        },
        {
          id: 'Flow_' + generateUUID(),
          name: '其它情况',
          showText: '未满足其它条件时，将进入此分支',
          type: NodeType.CONDITION_NODE,
          childNode: undefined,
          defaultFlow: true
        }
      ]
    }
    emits('update:childNode', data)
  }
  if (type === NodeType.DELAY_TIMER_NODE) {
    const data: SimpleFlowNode = {
      id: 'Activity_' + generateUUID(),
      name: NODE_DEFAULT_NAME.get(NodeType.DELAY_TIMER_NODE) as string,
      showText: '',
      type: NodeType.DELAY_TIMER_NODE,
      childNode: props.childNode
    }
    emits('update:childNode', data)
  }
}
</script>

<style lang="scss" scoped></style>
