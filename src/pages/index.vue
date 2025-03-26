<template>
    <main class="d-flex">
        <aside class="mr-2 pa-2" style="width: 400px; max-width: 400px;">
            <v-text-field
                v-model="search"
                label="search"
                hide-details
                hide-spin-buttons
                clearable
                class="mb-2"
                density="compact"/>

            <div style="width: 100%;">
                <div v-if="dataLeft" class="d-flex justify-space-between mb-1 bg-surface-light pa-1" style="width: 100%;">
                    <div class="text-dots" style="max-width: 200px;">{{ dataLeft.name }}</div>
                    <div>
                        <v-btn
                            class="text-caption"
                            variant="tonal"
                            density="compact"
                            icon="mdi-arrow-left"
                            color="primary">
                        </v-btn>
                        <v-btn
                            class="ml-1 text-caption"
                            variant="tonal"
                            density="compact"
                            icon="mdi-arrow-right"
                            @click="setRight(dataLeft.id)"
                            color="default">
                        </v-btn>
                    </div>
                </div>

                <div v-if="dataRight" class="d-flex justify-space-between bg-surface-light pa-1" style="width: 100%;">
                    <div class="text-dots" style="max-width: 200px;">{{ dataRight.name }}</div>
                    <div>
                        <v-btn
                            class="text-caption"
                            variant="tonal"
                            density="compact"
                            icon="mdi-arrow-left"
                            @click="setLeft(dataRight.id)"
                            color="default">
                        </v-btn>
                        <v-btn
                            class="ml-1 text-caption"
                            variant="tonal"
                            density="compact"
                            icon="mdi-arrow-right"
                            color="primary">
                        </v-btn>
                    </div>
                </div>

                <v-divider v-if="dataLeft || dataRight" class="mt-2 mb-2"></v-divider>

                <div style="max-height: 86vh; overflow-y: auto;">
                    <div v-for="d in dataMaches" :key="d.id" class="d-flex justify-space-between pa-1" style="width: 100%;">
                        <div class="text-dots" style="max-width: 200px;">{{ d.name }}</div>
                        <div>
                            <v-btn
                                class="text-caption"
                                variant="tonal"
                                density="compact"
                                icon="mdi-arrow-left"
                                @click="setLeft(d.id)"
                                :color="data.left === d.id ? 'primary':'default'">
                            </v-btn>
                            <v-btn
                                class="ml-1 text-caption"
                                variant="tonal"
                                density="compact"
                                icon="mdi-arrow-right"
                                @click="setRight(d.id)"
                                :color="data.right === d.id ? 'primary':'default'">
                            </v-btn>
                        </div>
                    </div>
                </div>
            </div>
        </aside>
        <div ref="el" style="width: 100%;">
            <div>
                <div class="d-flex align-center pr-4">
                    <span style="width: 100px;">Month</span>
                    <v-slider v-model.number="month"
                        :step="1"
                        :ticks="{
                            1: 'Jan', 2: 'Feb', 3: 'Mar',
                            4: 'April', 5: 'May', 6: 'June',
                            7: 'July', 8: 'Aug', 9: 'Sep',
                            10: 'Oct', 11: 'Nov', 12: 'Dec'
                        }"
                        class="text-caption"
                        show-ticks="always"
                        hide-details
                        hide-spin-buttons
                        @update:model-value="loadDetails"
                        :min="1"
                        :max="12"/>
                </div>
                <div class="d-flex align-center pr-4">
                    <span style="width: 100px;">Year</span>
                    <v-slider v-model.number="year"
                        :step="1"
                        show-ticks="always"
                        :ticks="[2020, 2021, 2022, 2023, 2024]"
                        class="text-caption"
                        hide-details
                        hide-spin-buttons
                        @update:model-value="loadDetails"
                        :min="2020"
                        :max="2024"/>
                </div>
            </div>
            <div class="d-flex align-center justify-space-between" style="width: 99%;">
                <div v-if="dataLeft" style="text-align: center;">
                    <h3>{{ dataLeft.name }}</h3>
                    <div class="sides" :style="{ height: (wmiddle*1.5)+'px' }">
                        <SunBurst v-if="data.sunLeft"
                            :data="data.sunLeft"
                            :width="wsides"
                            :height="wsides"
                            value-attr="value"
                            label-attr="name"/>

                        <div class="d-flex flex-column justify-center">
                            <div>Number of Cases</div>
                            <LineChart
                                :data="dataLeft.values"
                                :width="wsides"
                                :height="wsides*0.3"
                                :position="currentTime"
                                x-attr="Refdatum"
                                :y-attrs="['AnzahlFall']"/>
                        </div>

                        <div class="d-flex flex-column justify-center mt-2">
                            <div>Number of Deaths</div>
                            <LineChart
                                :data="dataLeft.values"
                                :width="wsides"
                                :height="wsides*0.3"
                                :position="currentTime"
                                x-attr="Refdatum"
                                :y-attrs="['AnzahlTodesfall']"/>
                        </div>
                    </div>
                </div>

                <GeoMap v-if="filtered"
                    :data="geo"
                    :values="filtered"
                    :highlights="highlights"
                    :width="wmiddle"
                    :height="wmiddle*1.5"/>

                <div v-if="dataRight" style="text-align: center;">
                    <h3>{{ dataRight.name }}</h3>
                    <div class="sides" :style="{ height: (wmiddle*1.5)+'px' }">
                        <SunBurst v-if="data.sunRight"
                            :data="data.sunRight"
                            :width="wsides"
                            :height="wsides"
                            value-attr="value"
                            label-attr="name"/>

                        <div class="d-flex flex-column justify-center">
                            <div>Number of Cases</div>
                            <LineChart
                                :data="dataRight.values"
                                :width="wsides"
                                :height="wsides*0.3"
                                :position="currentTime"
                                x-attr="Refdatum"
                                :y-attrs="['AnzahlFall']"/>
                        </div>

                        <div class="d-flex flex-column justify-center mt-2">
                            <div>Number of Deaths</div>
                            <LineChart
                                :data="dataRight.values"
                                :width="wsides"
                                :height="wsides*0.3"
                                :position="currentTime"
                                x-attr="Refdatum"
                                :y-attrs="['AnzahlTodesfall']"/>
                        </div>
                    </div>
                </div>
            </div>
        </div>

    </main>
</template>

<script setup>
    // import shp from 'shpjs'
    import { csv, json, autoType, group } from "d3"
    import GeoMap from '@/components/GeoMap.vue'
    import { computed, reactive } from "vue"
    import SunBurst from "@/components/SunBurst.vue"
    import LineChart from "@/components/LineChart.vue"
    import { useElementSize } from "@vueuse/core"

    const el = ref(null)
    const elSize = useElementSize(el)

    const wmiddle = computed(() => Math.max(200, elSize.width.value*0.4-60))
    const wsides = computed(() => Math.max(200, elSize.width.value*0.3))

    const year = ref(2022)
    const month = ref(3)
    const search = ref("")

    const INIT_LEFT = "Mannheim"
    const INIT_RIGHT = "Leipzig"

    // load data
    const data = reactive({
        main: [],
        sunLeft: null,
        sunRight: null,
        left: null,
        right: null
    })
    const dataLeft = computed(() => {
        if (data.left === null) {
            return null
        }
        return data.main.find(d => d.id === data.left)
    })
    const dataRight = computed(() => {
        if (data.right === null) {
            return null
        }
        return data.main.find(d => d.id === data.right)
    })
    const highlights = computed(() => [data.left, data.right].filter(d => d !== null))

    const dataMaches = computed(() => {
        const f = data.main.filter(d => d.id !== data.left && d.id !== data.right)
        if (!search.value) return f
        const regex = new RegExp(search.value, "gi")
        return f.filter(d => regex.test(d.name))
    })
    const currentTime = computed(() => new Date(year.value, month.value-1))

    const filtered = computed(() => {
        if (data.main.length === 0) return null
        const obj = {}
        data.main.forEach(d => {
            const it = d.values.find(dd => dd.Refdatum.getMonth() === month.value-1 && dd.Refdatum.getFullYear() === year.value)
            obj[d.id] = it ? it.AnzahlFall : 0
        })
        return obj
    })
    const geo = ref(null)

    function setLeft(id) {
        if (data.right === id) {
            data.right = null;
        }
        data.left = id;
        loadDetails()
    }
    function setRight(id) {
        if (data.left === id) {
            data.left = null;
        }
        data.right = id;
        loadDetails()
    }

    async function loadDetails() {
        let tmp = await csv("data/corona-monthly-details.csv", autoType)
        tmp = tmp.filter(d => d.Refdatum.getMonth() === month.value-1 && d.Refdatum.getFullYear() === year.value)
        if (data.left !== null) {
            data.sunLeft = makeTree(tmp.filter(d => d.IdLandkreis == data.left))
        } else {
            data.sunLeft = null
        }

        if (data.right !== null) {
            data.sunRight = makeTree(tmp.filter(d => d.IdLandkreis == data.right))
        } else {
            data.sunRight = null
        }
    }

    function makeTree(tmp) {
        const tree = {
            name: "",
            children: [],
        }

        const bySex = group(tmp, d => d.Geschlecht)
        bySex.forEach((array1, sex) => {
            const obj = {
                name: sex,
                children: [],
            }
            const byAge = group(array1, d => d.Altersgruppe)
            byAge.forEach((array2, ageGroup) => {
                const numFall = array2.reduce((acc, dd) => acc + dd.AnzahlFall, 0)
                const numDead = array2.reduce((acc, dd) => acc + dd.AnzahlTodesfall, 0)
                const numGen = array2.reduce((acc, dd) => acc + dd.AnzahlGenesen, 0)
                obj.children.push({
                    name: ageGroup,
                    children: [
                        { name: "Recovered", value: numFall },
                        { name: "Died", value: numDead },
                        { name: "Rest", value: numGen, hidden: true },
                    ]
                })
            })
            tree.children.push(obj)
        })

        return tree
    }

    async function init() {
        const map = await json("data/germany.json")
        const idx = new Map()
        map.objects.counties.geometries.forEach((d, i) => {
            d.id = +d.id
            idx.set(d.id, i)
        })
        geo.value = map

        const resp = await csv("data/corona-monthly.csv", autoType)
        const g = group(resp, d => d.IdLandkreis)

        const regL = new RegExp(INIT_LEFT, "gi")
        const regR = new RegExp(INIT_RIGHT, "gi")

        const list = []
        g.forEach((array, id) => {
            const i = idx.get(id)
            if (i === undefined) return
            const dt = map.objects.counties.geometries[i].properties.districtType;
            const n = map.objects.counties.geometries[i].properties.name
            list.push({
                id: id,
                name: n + ` (${dt})`,
                values: array
            })

            if (!data.left && regL.test(n)) {
                data.left = id
            }
            if (!data.right && regR.test(n)) {
                data.right = id
            }

        })
        list.sort((a, b) => {
            if (a.name < b.name) {
                return -1
            } else if (a.name > b.name) {
                return 1
            }
            return 0
        })
        data.main = list

        loadDetails()
    }

    onMounted(init)
</script>

<style>
.sides {
    background-color: #eee;
    text-align: center;
    border-radius: 4px;
    padding: 4px;
}

.text-dots {
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
}
</style>
