<template>
    <div v-if="!tvGuide && loading">
        <span class="loading loading-spinner loading-lg"></span>
    </div>

    <div v-if="tvGuide" class="p-4">
        <div class="flex mb-5 gap-3 text-3xl text-center">
            <div class="flex-none w-32"></div>
            <div class="flex-1 w-64 p-2 border rounded-md">What's on now</div>
            <div class="flex-1 w-64 p-2 border rounded-md">What's on next</div>
        </div>

        <div v-for="channel in tvGuide" class="flex mb-5 gap-3">
            <div class="flex-none w-32">
                <div v-if="channel.name == '9HD'">
                    <img src="@/assets/logos/nine.png" />
                </div>
                <div v-if="channel.name == 'ABC TV HD'">
                    <img src="@/assets/logos/abc.png" />
                </div>
                <div v-if="channel.name == 'SBS'">
                    <img src="@/assets/logos/sbs.png" />
                </div>
                <div v-if="channel.name == 'SBS Food'">
                    <img src="@/assets/logos/sbs-food.png" />
                </div>
                <div v-if="channel.name == '7HD'">
                    <img src="@/assets/logos/seven.png" />
                </div>
                <div v-if="channel.name == '10 HD'">
                    <img src="@/assets/logos/ten.png" />
                </div>
                {{ channel.name }}
            </div>
            <div v-for="item in channel.airings" class="flex-1 w-64 p-2 border rounded-md">
                <div class="font-bold text-2xl flex justify-between">
                    <span>{{ item.title }}</span>
                    <span>{{ formatTime(item.date) }}&ndash;{{ calculateFinishTime(item.date, item.duration) }}</span>
                </div>
                <div class="text-xl">{{ item.synopsis }}</div>
                <div v-if="item.remaining" class="mt-2">
                    <progress class="progress w-56 mr-10" :value="item.percent" max="100" />
                    <span>{{ item.remaining }} min remaining</span>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const tvGuide = ref(null)
const channelsArray = ["9HD", "10 HD", "ABC TV HD", "SBS", "SBS Food", "7HD"]
const loading = ref(false)

function calculateFinishTime(start, duration) {
    let startDate = new Date(start)
    console.log(startDate.getTime())
    return formatTime(new Date(startDate.getTime() + (duration * 60 * 1000)))
}

const formatTime = (input) => {
    let options = { hour: "numeric", minute: "numeric", hour12: true };
    let dt = new Date(input)
    return dt.toLocaleTimeString("en-GB", options);
}

const formatDate = (input) => {
    let options = { year: "numeric", month: "numeric", day: "numeric" };
    let dt = new Date(input)
    return dt.toLocaleDateString("en-GB", options);
}

const formatDateTime = (input) => {
    let options = { year: "numeric", month: "numeric", day: "numeric", hour: "2-digit", minute: "2-digit" };
    let dt = new Date(input)
    return dt.toLocaleDateString("en-GB", options);
}

async function getNowNext() {
    loading.value = true;
    const response = await fetch(" https://www.yourtv.com.au/api/now-next/?format=json&timezone=Australia%2FMelbourne&region=94")
    const json = await response.json()

    tvGuide.value = json[0].channels.filter(item => channelsArray.includes(item.name))
    loading.value = false;
}

onMounted(() => {
    getNowNext();
    const interval = setInterval(getNowNext, 30000)
})

</script>