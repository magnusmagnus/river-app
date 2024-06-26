<template>
    <li v-if="riverLevel !== 'Unable to get data'" class=" bg-slate-100 rounded-lg p-4 min-h-52 max-w-md">
        <div v-if="!loadingState">
            <div v-if="currentRiver">
                <p class="text-xl font-bold">{{ currentRiver.items.label }}</p>
                <p>{{ currentRiver.items.riverName }}</p>
                <div>
                    <div class="mt-2 text-xl ">
                        <p class="data-name">Status |
                            <span class="font-bold"
                                :class="levelStatus == 'Normal' ? 'text-green-700' : 'text-red-700'">{{
        levelStatus }}</span>
                        </p>
                    </div>
                    <div class="border border-black-300 mt-3 mb-2"></div>
                    <div>
                        <div>
                            <p class="data-name">Current Height |
                                <span>{{ riverLevel }}</span>
                            </p>
                        </div>
                        <div>
                            <p class="data-name">Typical Range |
                                <span>{{ normalRange }}</span>
                            </p>

                        </div>
                    </div>
                    <div class="border border-black-300 mt-3 mb-2"></div>
                    <div class="text-sm text-gray-500">
                        <p>Latest reading
                            <span>{{ lastUpdated ? lastUpdated : 'Unable to get data' }}</span>
                        </p>
                    </div>
                </div>
            </div>
        </div>
        <div v-else role="status" class="flex h-full justify-center items-center">
            <svg aria-hidden="true" class="w-8 h-8 text-gray-200 animate-spin dark:text-gray-600 fill-blue-600"
                viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path
                    d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z"
                    fill="currentColor" />
                <path
                    d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z"
                    fill="currentFill" />
            </svg>
            <span class="sr-only">Loading...</span>
        </div>
    </li>
</template>

<script setup>
const props = defineProps(['river', 'loadingState'])
const { river } = toRefs(props);

const riverUrl = river.value.replace('http://', 'https://')

const fetchRiverData = async () => {

    const { data } = await useFetch(riverUrl)

    return { data }
}

const { data: currentRiver } = await fetchRiverData()

const riverLevel = computed(() => {
    if (currentRiver.value.items.measures.latestReading) {
        return currentRiver.value.items.measures.latestReading.value + 'm'
    } else if (currentRiver.value.items.measures[1].latestReading.value) {
        return currentRiver.value.items.measures[1].latestReading.value + 'm'
    } else {
        return 'Unable to get data'
    }
})

const levelStatus = computed(() => {
    if (riverLevel >= currentRiver.value.items.stageScale.typicalRangeHigh) {
        return 'High'
    } else {
        return 'Normal'
    }
})

const normalRange = computed(() => {
    return currentRiver.value.items.stageScale.typicalRangeLow +
        'm to ' +
        currentRiver.value.items.stageScale.typicalRangeHigh +
        'm'
})

const lastUpdated = computed(() => {
    let lastUpdated = ''
    if (!currentRiver.value.items.measures[1]) {
        lastUpdated = currentRiver.value.items.measures.latestReading.dateTime
    } else {
        lastUpdated = currentRiver.value.items.measures[1].latestReading.dateTime
    }

    const timeStamp = new Date(lastUpdated)

    const dateToday = new Date()

    let dateTime

    if (isNaN(timeStamp)) {
        dateTime = 'Unable to get time'
        return dateTime
    }

    if (timeStamp.getDate() === dateToday.getDate()) {

        dateTime =
            'today at ' +
            timeStamp.getHours() +
            ':' +
            (timeStamp.getMinutes() < 10 ? '0' : '') +
            timeStamp.getMinutes()
    } else {
        dateTime =
            timeStamp.getDate() +
            '/' +
            (timeStamp.getMonth() + 1) +
            '/' +
            timeStamp.getFullYear() +
            ' at ' +
            timeStamp.getHours() +
            ':' +
            (timeStamp.getMinutes() < 10 ? '0' : '') +
            timeStamp.getMinutes()
    }

    return dateTime

})


</script>
