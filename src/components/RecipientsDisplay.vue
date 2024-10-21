<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import RecipientsBadge from '@/components/RecipientsBadge.vue'

const props = defineProps<{
  recipients: string[]
}>()

const recipientsRef = ref()
const badgeRef = ref()
const numTruncated = ref(props.recipients.length)
const tooltipMsg = ref('')
const isHover = ref(false)

onMounted(() => {
  showRecipientsList()
  window.addEventListener('resize', onResizeHandler)
})

onUnmounted(() => {
  window.removeEventListener('resize', onResizeHandler)
})

function getTextContent(value: string, index: number) {
  if (index < props.recipients.length - 1) return `${value}, `
  return value
}

function addSpanElement(textContent: string) {
  const newSpanElement = document.createElement('span')
  newSpanElement.textContent = textContent
  recipientsRef.value.appendChild(newSpanElement)
}

function removeSpanElement(index: number) {
  recipientsRef.value.removeChild(recipientsRef.value.children[index])
}

function isOverflow() {
  return recipientsRef.value.scrollWidth > recipientsRef.value.offsetWidth
}

function updateBadgeCount(count: number) {
  const spanElement = badgeRef.value.children[0]
  spanElement.textContent = `+${count}`
}

function handleOverflow(index: number, recipientsLength: number) {
  let childrenCount = recipientsRef.value.children.length
  // remove last element
  removeSpanElement(childrenCount - 1)

  // add ellipsis
  addSpanElement('...')

  // after ellipsis were added but it still overflows
  const isFirstChildindex = childrenCount - 2 === 0
  if (!isFirstChildindex && isOverflow()) {
    // remove added element before ellipsis
    removeSpanElement(childrenCount - 2)
    // update badge count
    updateBadgeCount(recipientsLength - --index)
  }
  return index
}

function removeAllRecipients() {
  while (recipientsRef.value.firstChild) {
    recipientsRef.value.removeChild(recipientsRef.value.firstChild)
  }
}

function showRecipientsList() {
  const recipientsLength = props.recipients.length
  let index = 0
  for (let value of props.recipients) {
    // update badge count for the number of truncated recipients
    updateBadgeCount(recipientsLength - index)

    // add span element for the current recipient value
    addSpanElement(getTextContent(value, index))

    // if an overflow happens after first element was added,
    // handle overflow then immediately break loop
    if (index > 0 && isOverflow()) {
      index = handleOverflow(index, recipientsLength)
      break
    }
    // increment index
    index++
  }
  numTruncated.value = recipientsLength - index
}

function onResizeHandler() {
  numTruncated.value = 0
  removeAllRecipients()

  showRecipientsList()
}

function handleMouseOver() {
  if (!isHover.value) {
    let index = 0
    let recipientList = ''
    for (let value of props.recipients) {
      recipientList += getTextContent(value, index)
      index++
    }

    tooltipMsg.value = recipientList
    isHover.value = true
  }
}

function handleMouseLeave() {
  isHover.value = false
}
</script>

<template>
  <div class="container">
    <div ref="recipientsRef" class="recipient-list"></div>
    <div v-show="numTruncated > 0" ref="badgeRef">
      <RecipientsBadge
        :numTruncated="numTruncated"
        @mouseleave="handleMouseLeave"
        @mouseover="handleMouseOver"
        class="recipient-badge"
      />
    </div>
    <span v-show="isHover" class="recipient-tooltip">
      {{ tooltipMsg }}
    </span>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  align-items: center;
  justify-content: space-between;
  column-gap: 8px;
  flex-wrap: wrap;
}
.recipient-list {
  flex-basis: 0;
  max-width: 100%;
  width: 100%;
  flex-grow: 1;
  overflow: hidden;
  text-align: left;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.recipient-badge {
  cursor: pointer;
}
.recipient-tooltip {
  position: fixed;
  top: 0;
  right: 0;
  display: flex;
  align-items: center;
  border-radius: 24px;
  color: #f0f0f0;
  background-color: #666;
  padding: 8px 16px;
  margin-top: 8px;
  margin-right: 8px;
}
</style>
