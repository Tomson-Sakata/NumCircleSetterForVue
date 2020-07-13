<template>
    <div style="width:100%; height:100%">
        <span>TEST</span><br/>
        <span v-if="controller != null">{{controller.value}}</span><br/>
        <canvas ref="canvas" width="200" height="200" style="width:100%; height:100%" />
    </div>
</template>

<script>
    // =========================================================
    // 位置を表すクラス
    class Position {
        constructor (x, y) {
            this.x = x
            this.y = y
        }
    }

    // =========================================================
    // コントローラーを表すクラス
    class Controller {
        constructor (canvas, centerX, centerY, radius, innerRadius, markerRadius, value) {
            this.self = this
            this.canvas = canvas
            this.radius = radius
            this.innerRadius = innerRadius
            this.center = new Position (centerX, centerY)
            this.markerRadius = markerRadius
            this.markerDistanceFromCenter = (radius - innerRadius) / 2 + innerRadius
            this.scaling = 1
            this.state = "idle"
            this.context = canvas.getContext("2d")
            this.value = value
            this.drawMarker()
            
            const self = this
            this.canvas.addEventListener("mousedown", function (e) {
                if (self.state != "idle") return
                self.onMoveMarker(e.clientX, e.clientY)
                self.state = "moving"
            })
            this.canvas.addEventListener("touchstart", function(e){
                if (self.state != "idle") return
                if (e.changedTouches.length == 1) {
                    e.preventDefault()
                    self.onMoveMarker(e.changedTouches[0].clientX, e.changedTouches[0].clientY)
                    self.state = "moving"
                }
            })
            this.canvas.addEventListener("mousemove", function (e) {
                if (self.state == "idle") return
                self.onMoveMarker(e.clientX, e.clientY)
            })
            this.canvas.addEventListener("touchmove", function (e) {
                if (self.state == "idle") return
                self.onMoveMarker(e.changedTouches[0].clientX, e.changedTouches[0].clientY)
            })
            this.canvas.addEventListener("mouseup", function (e) {
                if (self.state == "idle") return
                self.state = "idle"
                self.onMoveMarker(e.clientX, e.clientY)
            })
            this.canvas.addEventListener("touchend", function (e) {
                if (self.state == "idle") return
                self.state = "idle"
                self.onMoveMarker(e.changedTouches[0].clientX, e.changedTouches[0].clientY)
            })
            window.addEventListener("resize", function(){
                self.refreshScaling()
            })
            this.refreshScaling()
        }
        onMoveMarker (clientX, clientY) {
            const p = this.getPosition(clientX, clientY)
            const deg = this.getAngle(p.x, p.y)
            const value = deg / 360
            console.log(clientX, clientY, p.x, p.y, "Deg=" + deg, "Value=" + value)
            this.value = value
            this.drawMarker()
        }
        drawBase () {
            this.context.clearRect(0, 0, this.canvas.width, this.canvas.height)

            this.context.beginPath()
            this.context.fillStyle = "rgba(138,197,255,1)"
            this.context.arc(this.center.x, this.center.y, this.radius, 0, this.toRadian(360), false)
            this.context.fill()

            if (this.value > 0) {
                this.context.beginPath()
                this.context.fillStyle = "rgba(255,0,0,1)"
                this.context.moveTo(this.center.x, this.center.y)
                this.context.arc(this.center.x, this.center.y, this.radius, this.toRadian(270), this.toRadian(270 + 360 * this.value), false)
                this.context.fill()
            }

            this.context.beginPath()
            this.context.lineWidth = 1
            this.context.globalCompositeOperation = "destination-out"
            this.context.fillStyle = "rgba(0,0,0,1)"
            this.context.arc(this.center.x, this.center.y, this.innerRadius, 0, this.toRadian(360), false)
            this.context.fill()
            this.context.globalCompositeOperation = "source-over"
            
            this.context.beginPath()
            this.context.lineWidth = 2
            this.context.strokeStyle = "rgba(0,0,0,1)"
            this.context.arc(this.center.x, this.center.y, this.radius, 0, this.toRadian(360), false)            
            this.context.stroke()
            
            this.context.beginPath()
            this.context.lineWidth = 2
            this.context.strokeStyle = "rgba(0,0,0,1)"
            this.context.arc(this.center.x, this.center.y, this.innerRadius, 0, this.toRadian(360), false)
            this.context.stroke()

            this.context.beginPath()
            this.context.save()
            this.context.translate(this.center.x, this.center.y)
            this.context.rotate(this.toRadian(180))
            this.context.moveTo(0, this.innerRadius)
            this.context.lineTo(0, this.radius)
            this.context.stroke()
            this.context.restore()
        }
        drawMarker () {
            this.drawBase()
            const deg = (1 * this.value) * 360 
            this.context.save()
            this.context.translate(this.center.x, this.center.y)
            this.context.rotate(this.toRadian(deg))

            if (this.state == "moving") {
                this.context.beginPath()
                this.context.fillStyle = "rgba(25,118,210,0.5)"
                this.context.arc(0, -this.markerDistanceFromCenter, this.markerRadius * 2, 0, this.toRadian(360), false)
                this.context.fill()
            }

            this.context.beginPath()
            this.context.strokeStyle = "rgba(0,0,0,1)"
            this.context.moveTo(0, -this.innerRadius - 1)
            this.context.lineTo(0, -this.radius + 1)
            this.context.stroke()
            
            this.context.beginPath()
            this.context.fillStyle = "rgba(25,118,210,1)"
            this.context.strokeStyle = "rgba(0,0,0,1)"
            this.context.arc(0, -this.markerDistanceFromCenter, this.markerRadius, 0, this.toRadian(360), false)
            this.context.fill()
            // this.context.stroke()
            this.context.restore()
        }
        refreshScaling () {
            this.scaling = this.canvas.clientWidth / this.canvas.width
            console.log("refreshScaling", this.canvas.clientWidth, this.canvas.width, this.scaling)
        }
        getPosition (x, y) {
            const rect = this.canvas.getBoundingClientRect()
            return new Position((x - rect.left) / this.scaling, (y - rect.top) / this.scaling)
        }
        getAngle (x, y) {
            var deg = Math.atan2(y - this.center.y, x - this.center.x) * ( 180 / Math.PI ) + 90
            if (deg < 0) deg = 360 + deg
            if (deg > 360) deg -=360
            return deg
        }
        toRadian (angle) {
            if (angle == 0) {
                return 0
            }
            return angle * Math.PI / 180
        }
    }

    export default {
        data: function () {
            return {
                controller: null,
            }
        },
        mounted: function () {
            this.controller = new Controller(this.$refs.canvas, 100, 100, 98, 50, 10, 0)
        },
        methods: {
        }
    }
</script>
