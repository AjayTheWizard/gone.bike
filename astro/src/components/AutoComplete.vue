<script setup lang="ts">
import { computed, nextTick, onBeforeMount, onMounted, ref, toRaw, watch } from "vue"
import { t } from 'i18next'
import { onClickOutside } from "@vueuse/core"
import isMobile from "ismobilejs";

const props = defineProps<{ api: { key: string | number, value: string }[], title: string, listed: string, newItem: string, topOffset: number }>()
const emit = defineEmits(['update:listed', 'update:newItem', 'update:topOffset'])
const isOpen = ref(false)
let startIdx = ref(0)
function handleChange(e: Event) {
    if (!firstClick.value) {
        firstClick.value = true
    }
    let val = (e.target as HTMLInputElement).value
    focusElIdx.value = 0
    searchQuery.value = val
}
const searchEl = ref<HTMLInputElement>()
const prevQuery = ref<string[]>([])
const searchQuery = ref('')
const queryResults = computed(() => {
    prevQuery.value = queryResults.value
    if (props.api.length == 0) {
        return [ searchQuery.value ];
    }
    let tempresult = props.api.map(e => e.value).filter(q => q.toLowerCase().includes(searchQuery.value.toLowerCase()))
    let result = startIdx.value === 0 ? tempresult : tempresult.slice(startIdx.value, tempresult.length - 1)
    if (result.length === 0) {
        result.push(searchQuery.value)
    }
    return result
})
let firstClick = ref(false)

watch(firstClick, function (first) {
    if (first && (isMobile(navigator.userAgent).phone || isMobile(navigator.userAgent).tablet)) {
        if (props.title === "bike_brand") {
            if (props.topOffset === 0) {
                // console.log(document.getElementById(`dropdown-label-${props.title}`)!.getClientRects().item(0)!.top)
                emit("update:topOffset", document.getElementById(`dropdown-label-${props.title}`)!.getClientRects().item(0)!.top)
                window.scrollTo({
                    top: document.getElementById(`dropdown-label-${props.title}`)!.getClientRects().item(0)!.top,
                    behavior: "smooth"
                })
            } else {
            }
            // console.log(props.topOffset)
        }else{
            window.scrollTo({
                top: props.topOffset + 150,
                behavior: "smooth"
            })
        }
    }
})

onMounted(() => {
    /**
     * Template ref wa avoided because It would have access to same element at All Instance of Autocomplete Component
     */
    searchEl.value = document.getElementById(`dropdown-${props.title}`) as HTMLInputElement
})
onClickOutside(searchEl, function () {
    if (firstClick.value) {
        firstClick.value = false
    }
    isOpen.value = false
})
const focusElIdx = ref(0)
watch(focusElIdx, function (idx) {
})
function searchKeyDown() {
    if (queryResults.value.length === 1) {
        focusElIdx.value = 0
    } else if (queryResults.value.length === (focusElIdx.value + 1)) {
    } else if (queryResults.value.length > (focusElIdx.value + 1)) {
        if (focusElIdx.value + 1 > 5) {
            startIdx.value++
        }
        focusElIdx.value = Math.min(5, focusElIdx.value + 1)
    }
}
function searchKeyUp() {
    if (queryResults.value.length === 1 || focusElIdx.value === 0) {

    } else if (queryResults.value.length >= (focusElIdx.value + 1)) {
        if (startIdx.value !== 0) {
            startIdx.value--
            focusElIdx.value--
        } else {
            focusElIdx.value--
        }
    }
}
function searchEnter(result?: string) {
    if (firstClick.value) {
        firstClick.value = false
    }
    if (result) {
        searchQuery.value = result
    } else {
        searchQuery.value = queryResults.value[focusElIdx.value] ?? prevQuery.value[focusElIdx.value]
    }
    searchEl.value?.blur()
    isOpen.value = false
    let apiList = props.api.map(e => e.value)
    let idx = apiList.indexOf(result ? result : (queryResults.value[focusElIdx.value] ?? prevQuery.value[focusElIdx.value]))
    if (idx > -1) {
        emit("update:listed", props.api[idx].key)
        emit("update:newItem", "")
    } else {
        emit("update:listed", "")
        emit("update:newItem", searchQuery.value)
    }
}
</script>
<template>
    <div class="relative w-full h-full">
        <div class="flex flex-col relative w-full">
            <label :id="`dropdown-label-${props.title}`" :for="props.title" class='mb-2 text-lg w-full'>{{
                t(`forms.report.questions.${props.title}.title`)
            }}</label>
            <div :id="`dropdown-${props.title}`" class="w-full">
                <input @focus="firstClick = true" :id="`dropdown-label-${props.title}`"
                    :placeholder="(t(`forms.report.questions.${props.title}.placeholder`) as string)" class="z-10 w-full report-placeholder"
                    @keydown.enter="function () { searchEnter() }" @keydown.arrow-down="searchKeyDown"
                    @keydown.arrow-up="searchKeyUp" @focusin="isOpen = true" :value="searchQuery" type="search" autocomplete="off"
                    @input="handleChange">
                <div
                    :class="['border-gray-400 mt-1 border top-26 z-50 absolute w-full h-fit max-h-64 overflow-y-scroll bg-white', { 'hidden': !isOpen }]">
                    <div v-for="(result, idx) in queryResults" id="here"
                        :class="['p-2 border-b-2 hover:bg-blue-200 flex gap-2 items-center cursor-pointer border-gray-300', { 'bg-blue-200': idx === focusElIdx, 'font-semibold': result === searchQuery }]"
                        :key="result" @click="searchEnter(result)">
                        {{ result }}
                    </div>
                </div>
            </div>
            <span class="text-sm mt-2 font-normal italic text-gray-400">{{
                t(`forms.report.questions.${props.title}.subtitle`)
            }}</span>
        </div>
    </div>
</template>
<style>
[type='text'],
[type='email'],
[type='url'],
[type='password'],
[type='number'],
[type='date'],
[type='datetime-local'],
[type='month'],
[type='search'],
[type='tel'],
[type='time'],
[type='week'],
[multiple],
textarea,
select {
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
    background-color: #fff;
    border-color: #6b7280 !important;
    border-width: 1px;
    border-radius: 0px !important;
    padding-top: 0.5rem;
    padding-right: 0.75rem;
    padding-bottom: 0.5rem;
    padding-left: 0.75rem;
    font-size: 1rem;
    line-height: 1.5rem;
    --tw-shadow: 0 0 #0000;
}
</style>
