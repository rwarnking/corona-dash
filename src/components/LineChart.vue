<template>
    <svg ref="el" :width="width" :height="height"></svg>
</template>
<script setup>
    import * as d3 from 'd3'
    import { onMounted } from 'vue'

    const props = defineProps({
        data: {
            type: Object,
            required: true
        },
        position: {
            type: Date,
            required: false
        },
        width: {
            type: Number,
            default: 500,
        },
        height: {
            type: Number,
            default: 500,
        },
        xAttr: {
            type: String,
            default: "x"
        },
        yAttrs: {
            type: Array,
            default: () => (["y"])
        },
        labelAttr: {
            type: String,
        },
    })

    const el = ref(null)

    const getX = d => d[props.xAttr]
    const getY = (d, attr) => d[attr]

    let x, y, colors;

    function init() {
        const svg = d3.select(el.value)
        svg.selectAll("*").remove()

        const off = 25, margin = 10

        const area = {}
        const path = {}

        let allMax = 0;
        props.yAttrs.forEach(attr => {
            allMax = Math.max(allMax, d3.max(props.data, d => getY(d, attr)))
        })

        x = d3.scaleTime()
            // .domain([getX(data[0].values.at(0)), getX(data[0].values.at(-1))])
            .domain(d3.extent(props.data, getX))
            .range([margin+5+off, props.width-margin-5])

        y = d3.scaleLinear()
            .domain([0, allMax])
            .range([props.height-margin-off, margin])

        props.yAttrs.forEach(attr => {
            area[attr] = d3.area()
                .x(d => x(getX(d)))
                .y0(y(0))
                .y1(d => y(getY(d, attr)))

            path[attr] = d3.line()
                .curve(d3.curveMonotoneX)
                .x(d => x(getX(d)))
                .y(d => y(getY(d, attr)))
        })


        colors = d3.scaleOrdinal(d3.schemeObservable10)
            .domain(props.yAttrs)

        svg.selectAll(".area")
            .data(props.yAttrs)
            .join("path")
            .classed("area", true)
            .attr("d", d => area[d](props.data))
            .attr("fill", d => colors(d))
            .attr("stroke", "none")
            .attr("fill-opacity", 0.15)

        svg.selectAll(".line")
            .data(props.yAttrs)
            .join("path")
            .classed("line", true)
            .attr("d", d => path[d](props.data))
            .attr("fill", "none")
            .attr("stroke", d => colors(d))
            .attr("stroke-width", 1)

        svg.append("g")
            .attr("transform", `translate(0,${props.height-off-margin})`)
            .call(d3.axisBottom(x).ticks(Math.floor(props.width / 50)))

        svg.append("text")
            .attr("x", x.range()[0] - 5)
            .attr("y", 13)
            .attr("text-anchor", "end")
            .attr("font-size", 10)
            .text(d3.format(".2s")(allMax))

        svg.append("line")
            .attr("x1", x.range()[0])
            .attr("x2", x.range()[1])
            .attr("y1", margin)
            .attr("y2", margin)
            .attr("stroke", "lightgrey")

        drawPosition()
    }

    function drawPosition() {
        const svg = d3.select(el.value)
        svg.selectAll(".pos").remove()

        const pos = svg.selectAll(".pos")
            .data(props.position ? [{ id: 1, data: props.position }] : [])
            .join("g")
            .classed("pos", true)

        pos.append("line")
            .attr("x1", d => x(d.data))
            .attr("x2", d => x(d.data))
            .attr("y1", y.range()[0])
            .attr("y2", y.range()[1])
            .attr("stroke", "black")
            .attr("fill", "none")
            .attr("stroke-width", 2)
            .attr("stroke-dasharray", "4 2")

        pos.append("circle")
            .attr("cx", d => x(d.data))
            .attr("cy", y.range()[0])
            .attr("r", 3)
            .attr("stroke", "black")
            .attr("fill", colors(props.yAttrs[0]))
            .attr("stroke-width", 1)

    }

    onMounted(init)

    watch(() => props.position, drawPosition)
    watch(() => {
        const obj = Object.assign({}, props)
        delete obj.position
        return obj
    }, init, { deep: true })
</script>
