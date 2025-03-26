<template>
    <div>
        <div>
            <svg ref="el" :width="width" :height="height" :view-box="[0, 0, width, height]" style="max-width: 100%; height: auto;"></svg>
        </div>
        <div>
            <svg ref="legend" :width="width" :height="legendSize"></svg>
        </div>
    </div>
</template>

<script setup>

    import * as d3 from 'd3'
    import { feature, mesh, transform } from 'topojson'
    import { onMounted, watch } from 'vue'

    const props = defineProps({
        data: {
            type: Object,
            required: true
        },
        values: {
            type: Object,
            required: true
        },
        width: {
            type: Number,
            default: 400
        },
        height: {
            type: Number,
            default: 600
        },
        legendSize: {
            type: Number,
            default: 50
        },
        highlights: {
            type: Array,
            required: false
        }
    })

    const el = ref(null)
    const legend = ref(null)

    function draw() {
        if (!props.data || !props.values) return
        const geodata = feature(props.data, props.data.objects.counties)
        const color = d3.scaleQuantile(Object.values(props.values), d3.schemeOrRd[9])

        const [bottomLeft, topRight] = d3.geoBounds(geodata)

        const lambda = -(topRight[0] + bottomLeft[0]) / 2

        const center = [
            (topRight[0] + bottomLeft[0]) / 2 + lambda,
            (topRight[1] + bottomLeft[1]) / 2
        ]

        const scale = Math.min(
            props.width / (topRight[0] + bottomLeft[0]),
            props.height / (topRight[1] - bottomLeft[1])
        )

        const proj = d3.geoAlbers()
            .parallels([bottomLeft[1], topRight[1]])
            .translate([props.width / 2, props.height / 2])
            .rotate([lambda, 0, 0])
            .center(center)
            .scale(scale * 200)

        const path = d3.geoPath(proj)

        const svg = d3.select(el.value)
        svg.selectAll("*").remove()

        svg.append("g")
            .selectAll("path")
            .data(geodata.features)
            .join("path")
            .attr("d", path)
            .attr("fill", d => color(props.values[d.id]))

        svg.append("g")
            .append("path")
            .datum(mesh(props.data, props.data.objects.counties, (a, b) => a !== b))
            .attr("fill", "none")
            .attr("stroke", "white")
            .attr("stroke-linejoin", "round")
            .attr("d", path);

        if (props.highlights) {
            const f = geodata.features.filter(d => props.highlights.includes(d.id))
            const cs = f.map(d => path.centroid(d))

            svg.append("path")
                .attr("d", `M 0,50 L ${cs[0][0]},${cs[0][1]} L 0,${props.height}`)
                .attr("stroke", d3.schemeObservable10[0])
                .attr("stroke-width", 3)
                .attr("fill", "none")

            svg.append("path")
                .attr("d", `M ${props.width},50 L ${cs[1][0]},${cs[1][1]} L ${props.width},${props.height}`)
                .attr("stroke", d3.schemeObservable10[0])
                .attr("stroke-width", 2)
                .attr("fill", "none")

            svg.append("g")
                .selectAll("path")
                .data(f)
                .join("path")
                .attr("d", path)
                .attr("stroke", d3.schemeObservable10[0])
                .attr("stroke-width", 2)
                .attr("fill", d => color(props.values[d.id]))
        }

        drawLegend(color)
    }

    function drawLegend(color) {
        const svg = d3.select(legend.value)
        svg.selectAll("*").remove()

        const thresholds
            = color.thresholds ? color.thresholds() // scaleQuantize
            : color.quantiles ? color.quantiles() // scaleQuantile
            : color.domain(); // scaleThreshold

        const margin = 15

        const x = d3.scaleLinear()
            .domain([-1, color.range().length - 1])
            .rangeRound([5, props.width-5]);

        svg.append("g")
            .selectAll("rect")
            .data(color.range())
            .join("rect")
                .attr("y", 0)
                .attr("x", (_, i) => x(i-1))
                .attr("width", (_, i) => x(i)-x(i-1))
                .attr("height", margin)
                .attr("fill", d => d)

        const tickValues = d3.range(thresholds.length);

        svg.append("g")
            .attr("transform", `translate(0,${margin})`)
            .call(d3.axisBottom(x).tickValues(tickValues).tickFormat((_,i) => thresholds[i].toFixed(0)))
            .call(g => g.select(".domain").remove())

    }

    onMounted(draw)

    watch(props, draw)
</script>
