<template>
    <div>
        <Setter v-model="value1" :min=0 :max=1 :size=150 :innerRadiusRate="0.6" style="margin-bottom:10px" />
        <Setter v-model="value2" :min=5 :max=10 :circleColor="{r:178, g:235, b:242, a:1}" :circleValueColor="{r:0, g:151, b:167, a:1}" :markerColor="{r:255, g:0, b:0, a:1}" style="margin-bottom:10px" />
        <Setter v-model="value3">
            <span>{{value3.toFixed(2)}}</span>
        </Setter>
    </div>
</template>

<script>
    import Setter from "@/components/NumCircleSetter"
    export default {
        name: 'HelloWorld',
        components: {
            Setter
        },
        props: {
        },
        data: function () {
            return {
                value1: 0.7,
                value2: 8,
                value3: 0,
            }
        }
    }
</script>
