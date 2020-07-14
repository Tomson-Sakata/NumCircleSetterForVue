# NumCircleSetterForVue
A component for Vue.js that allows numbers to be specified on a circle.

# Requirement
Vue.js

# Screen shot
![画面](https://github.com/Tomson-Sakata/NumCircleSetter/blob/images/screenshot_1.jpg)

# Usage
1.copy src/components/NumCircleSetter.vue to your project.

2.Embedded in the components of your project.

    <template>
        <div>
            <Setter v-model="value1" :min=0 :max=1 :size=150 :innerRadiusRate="0.6" />
            <Setter v-model="value2" :min=5 :max=10 :circleColor="{r:178, g:235, b:242, a:1}" :circleValueColor="{r:0, g:151, b:167, a:1}" :markerColor="{r:255, g:0, b:0, a:1}" />
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
                    value1: 0,
                    value2: 5
                }
            }
        }
    </script>

# API

    Properties

        size ... Component Size(default:200px)
        innerRadiusRate ... Radius of the inner circle(default:0.5%)
        markerRadius ... Marker Radius(default:10px)
        min ... Minimum value(default:0)
        max ... Minimum value(default:1)
        markerColor ... Marker Color(default:{r:25, g:118, b:210, a:1})
        circleColor ... Color of the circle(default:{r:138, g:197, b:255, a:1})
        circleValueColor ... Color of the circle value(default:{r:255, g:0, b:0, a:1})

![画面](https://github.com/Tomson-Sakata/NumCircleSetter/blob/images/properties_1.jpg)
