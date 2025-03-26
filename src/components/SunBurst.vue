<template>
    <svg ref="el" :width="width" :height="height"></svg>
</template>
<script setup>
    import * as d3 from 'd3'
    import { onMounted, watch } from 'vue'

    const props = defineProps({
        data: {
            type: Object,
            required: true
        },
        width: {
            type: Number,
            default: 500,
        },
        height: {
            type: Number,
            default: 500,
        },
        idAttr: {
            type: String,
            default: "id"
        },
        parentAttr: {
            type: String,
            default: "parent"
        },
        valueAttr: {
            type: String,
            default: "value"
        },
        labelAttr: {
            type: String,
        },
    })

    const el = ref(null)

    const getV = d => d[props.valueAttr]
    const getL = d => d[props.labelAttr]

    function init() {
        const root = d3.hierarchy(props.data)

        root.sum(d => getV(d))

        const margin = 10, padding = 1
        const radius =  Math.min(props.width - margin * 2, props.height - margin * 2) / 2

        d3.partition().size([Math.PI * 2, radius])(root);

        // Construct an arc generator.
        const arc = d3.arc()
            .startAngle(d => d.x0)
            .endAngle(d => d.x1)
            .padAngle(d => Math.min((d.x1 - d.x0) / 2, 2 * padding / radius))
            .padRadius(radius / 2)
            .innerRadius(d => d.y0)
            .outerRadius(d => d.y1 - padding);

        const svg = d3.select(el.value)
            .attr("viewBox", [
                margin - margin - props.width / 2,
                margin - margin - props.height / 2,
                props.width,
                props.height
            ])
            .attr("style", "max-width: 100%; height: auto; height: intrinsic;")
            .attr("font-family", "sans-serif")
            .attr("font-size", 10)
            .attr("text-anchor", "middle")

        svg.selectAll("*").remove()

        const scales = {
            1: d3.scaleOrdinal(["#4269d0", "#97bbf5", "#ff7f0e"]).domain(["M", "W", "unbekannt"]),
            2: d3.scaleOrdinal(d3.schemeGreys[7].slice(1)).domain(["A00-A04", "A05-A14", "A15-A34", "A35-A59", "A60-A79", "A80+"]),
            3: d3.scaleOrdinal(["#2ca02c", "#d62728"]).domain(["Recovered", "Died"]),
        }

        const cell = svg
            .selectAll("a")
            .data(root.descendants())
            .join("a")

        const color = d3.scaleSequential([0, root.children.length], d3.interpolateRainbow).unknown("black");
        root.children.forEach((child, i) => child.index = i);

        cell.filter(d => !d.data.hidden && d.depth)
            .append("path")
            .attr("d", arc)
            .attr("fill", d => scales[d.depth](d.data.name))
            .attr("fill-opacity", 0.5)

        if (props.labelAttr) {
            cell
                .filter(d => !d.data.hidden && (d.y0 + d.y1) / 2 * (d.x1 - d.x0) > 10)
                .append("text")
                .attr("transform", d => {
                    if (!d.depth) return;
                    const x = ((d.x0 + d.x1) / 2) * 180 / Math.PI;
                    const y = (d.y0 + d.y1) / 2;
                    return `rotate(${x - 90}) translate(${y},0) rotate(${x < 180 ? 0 : 180})`;
                })
                .attr("dy", "0.32em")
                .text(d => getL(d.data));
        }
    }

    onMounted(init)

    watch(props, init, { deep: true })
</script>
