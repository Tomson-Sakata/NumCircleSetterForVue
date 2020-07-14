<template>
    <div :style="'width:'+size+'px; height:'+size+'px; position:relative; top:0, left:0'">
        <canvas ref="canvas" :width="size" :height="size" />
        <div :style="textAreaStyle" ref="textArea">  <!-- "'position:absolute; top:50%; left:50%; width:'+(innerRadius * 2)+'px'" ref="textArea"> -->
            <slot />
            <!-- <span :style="labelStyle">{{value.toFixed(2)}}</span>             -->
        </div>
    </div>
</template>

<script>
    class Position {
        constructor (x, y) {
            this.x = x
            this.y = y
        }
    }

    export default {
        props: {
            value: {
                type: Number,
                default: 0
            },
            size: {
                type: Number,
                default: 200
            },
            innerRadiusRate: {
                type: Number,
                default: 0.5
            },
            markerRadius: {
                type: Number,
                default: 10
            },
            min: {
                type: Number,
                default: 0
            },
            max: {
                type: Number,
                default: 1
            },
            markerColor: {
                type: Object,
                default: function () {
                    return {
                        r:25, g:118, b:210, a:1
                    }
                }
            },
            circleColor: {
                type: Object,
                default: function () {
                    return {
                        r:138, g:197, b:255, a:1
                    }
                }
            },
            circleValueColor: {
                type: Object,
                default: function () {
                    return {
                        r:255, g:0, b:0, a:1
                    }
                }
            },
        },
        computed: {
            returnValue: {
                get: function () {
                    return this.$props
                },
                set: function (v) {
                    this.$emit("input", v)
                }
            },
        },
        data: function () {
            return {
                canvas: null,
                context: null,
                center: null,
                radius: null,
                innerRadius: null,
                markerDistanceFromCenter: 0,
                scaling: 1,
                state: "idle",
                normalizedValue: 0,
                markerColorRGBA: "",
                markerEffectColorRGBA: "",
                circleColorRGBA: "",
                circleValueColorRGBA: "",
                textAreaStyle: ""
            }
        },
        mounted: function () {
            const self = this
            this.canvas = this.$refs.canvas
            this.context = this.canvas.getContext("2d")

            this.center = new Position(this.canvas.width / 2, this.canvas.height / 2)
            this.radius = this.canvas.width / 2
            this.innerRadius = this.radius * this.innerRadiusRate

            this.markerDistanceFromCenter = (this.radius - this.innerRadius) / 2 + this.innerRadius

            this.markerColorRGBA = "rgba("+this.markerColor.r+","+this.markerColor.g+","+this.markerColor.b+","+this.markerColor.a+")"
            this.markerEffectColorRGBA = "rgba("+this.markerColor.r+","+this.markerColor.g+","+this.markerColor.b+",0.5)"

            this.circleColorRGBA = "rgba("+this.circleColor.r+","+this.circleColor.g+","+this.circleColor.b+","+this.circleColor.a+")"
            this.circleValueColorRGBA = "rgba("+this.circleValueColor.r+","+this.circleValueColor.g+","+this.circleValueColor.b+","+this.circleValueColor.a+")"

            const textAreaPosition = this.getRotatedPosition(this.center.x, this.center.y, this.center.x, this.center.y - this.innerRadius, 360 - 45)
            const textAreaSize = ( this.center.x - textAreaPosition.x ) * 2
            this.textAreaStyle = {
                "position": "absolute",
                "display": "flex",
                "justify-content": "center",
                "align-items": "center",
                "top": textAreaPosition.y.toFixed(0) + "px",
                "left": textAreaPosition.x.toFixed(0) + "px",
                "width": textAreaSize + "px",
                "height": textAreaSize + "px",
            }

            this.canvas.addEventListener("mousedown", function (e) {
                if (self.state != "idle") return
                const p = self.getPosition(e.clientX, e.clientY)
                if (self.isOnMarker(p)) {
                    self.moveMarker(p)
                    self.state = "marker_moving"
                } else {
                    self.state = "area_touched"
                }
            })
            this.canvas.addEventListener("touchstart", function(e){
                if (self.state != "idle") return
                if (e.changedTouches.length == 1) {
                    e.preventDefault()
                    const p = self.getPosition(e.changedTouches[0].clientX, e.changedTouches[0].clientY)
                    if (self.isOnMarker(p)) {
                        self.moveMarker(p)
                        self.state = "marker_moving"
                    } else {
                        self.state = "area_touched"
                    }
                }
            })
            this.canvas.addEventListener("mousemove", function (e) {
                if (self.state == "idle" || self.state == "area_touched") return
                const p = self.getPosition(e.clientX, e.clientY)
                self.moveMarker(p)
            })
            this.canvas.addEventListener("touchmove", function (e) {
                if (self.state == "idle" || self.state == "area_touched") return
                const p = self.getPosition(e.changedTouches[0].clientX, e.changedTouches[0].clientY)
                self.moveMarker(p)
            })
            this.canvas.addEventListener("mouseup", function (e) {
                if (self.state == "idle") return
                const isAnimation = (self.state == "area_touched")
                self.state = "idle"
                const p = self.getPosition(e.clientX, e.clientY)
                self.moveMarker(p, isAnimation)
            })
            this.canvas.addEventListener("touchend", function (e) {
                if (self.state == "idle") return
                const isAnimation = (self.state == "area_touched")
                self.state = "idle"
                const p = self.getPosition(e.changedTouches[0].clientX, e.changedTouches[0].clientY)
                self.moveMarker(p, isAnimation)
            })
            window.addEventListener("resize", function(){
                self.refreshScaling()
            })
            this.refreshScaling()

            this.normalizedValue = (this.value - this.min) / (this.max - this.min) 
            console.log(this.normalizedValue)
            this.drawMarker(this.normalizedValue)
        },
        methods: {
            isOnMarker (p) {
                const image = this.context.getImageData(p.x, p.y, 1, 1)
                if (this.markerColor.r == image.data[0] && this.markerColor.g == image.data[1] && this.markerColor.b == image.data[2]) {
                    return true
                }
                return false
            },
            moveMarker (p, isAnimation) {
                const oldValue = this.normalizedValue
                const deg = this.getAngle(p.x, p.y)
                const normalizedValue = deg / 360
                this.normalizedValue = normalizedValue
                this.returnValue = (this.max - this.min) * this.normalizedValue + this.min
                if (isAnimation != null && isAnimation) {
                    const self = this
                    var from = oldValue
                    var to = this.normalizedValue
                    var animationTime = (Math.abs(to - from) / 1 ) * 200  // 0->1 に変化する時間を 0.2s とする
                    const startTime = performance.now()
                    requestAnimationFrame(function drawer(time){
                        const interval = time - startTime
                        var p = interval / animationTime
                        if (p >= 1) p=1
                        const v = (to - from) * p + from
                        self.drawMarker(v)
                        if (p < 1) requestAnimationFrame(drawer)
                    })
                } else {
                    this.drawMarker(this.normalizedValue)
                }
            },
            drawBase (value) {
                this.context.clearRect(0, 0, this.canvas.width, this.canvas.height)

                this.context.beginPath()
                this.context.fillStyle = this.circleColorRGBA
                this.context.arc(this.center.x, this.center.y, this.radius, 0, this.toRadian(360), false)
                this.context.fill()

                if (value > 0) {
                    this.context.beginPath()
                    this.context.fillStyle = this.circleValueColorRGBA
                    this.context.moveTo(this.center.x, this.center.y)
                    this.context.arc(this.center.x, this.center.y, this.radius, this.toRadian(270), this.toRadian(270 + 360 * value), false)
                    this.context.fill()
                }

                this.context.beginPath()
                this.context.lineWidth = 1
                this.context.globalCompositeOperation = "destination-out"
                this.context.fillStyle = "rgba(0,0,0,1)"
                this.context.arc(this.center.x, this.center.y, this.innerRadius, 0, this.toRadian(360), false)
                this.context.fill()
                this.context.globalCompositeOperation = "source-over"
            },
            drawMarker (value) {
                this.drawBase(value)
                const deg = (1 * value) * 360 
                this.context.save()
                this.context.translate(this.center.x, this.center.y)
                this.context.rotate(this.toRadian(deg))

                if (this.state == "marker_moving") {
                    this.context.beginPath()
                    this.context.fillStyle = this.markerEffectColorRGBA
                    this.context.arc(0, -this.markerDistanceFromCenter, this.markerRadius * 3, 0, this.toRadian(360), false)
                    this.context.fill()
                }
                
                this.context.beginPath()
                this.context.fillStyle = this.markerColorRGBA
                this.context.strokeStyle = "rgba(0,0,0,1)"
                this.context.arc(0, -this.markerDistanceFromCenter, this.markerRadius, 0, this.toRadian(360), false)
                this.context.fill()
                this.context.restore()
            },
            refreshScaling () {
                this.scaling = this.canvas.clientWidth / this.canvas.width
            },
            getPosition (x, y) {
                const rect = this.canvas.getBoundingClientRect()
                return new Position((x - rect.left) / this.scaling, (y - rect.top) / this.scaling)
            },
            getRotatedPosition (centerX, centerY, positionX, positionY, deg) {
                const r = this.toRadian(deg)
                const x = ((positionX - centerX) * Math.cos(r) - (positionY - centerY) * Math.sin(r)) + centerX
                const y = ((positionX - centerX) * Math.sin(r) + (positionY - centerY) * Math.cos(r)) + centerY
                return new Position(x, y)
            },
            getAngle (x, y) {
                var deg = Math.atan2(y - this.center.y, x - this.center.x) * ( 180 / Math.PI ) + 90
                if (deg < 0) deg = 360 + deg
                if (deg > 360) deg -=360
                return deg
            },
            toRadian (angle) {
                if (angle == 0) {
                    return 0
                }
                return angle * Math.PI / 180
            }
        }
    }
</script>
