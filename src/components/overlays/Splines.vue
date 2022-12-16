<script>
// Channel renderer. (Keltner, Bollinger)
import Overlay from '../../mixins/overlay.js'

export default {
    name: 'Splines',
    mixins: [Overlay],
    methods: {
        meta_info() {
            return { author: 'C451', version: '1.1.0' }
        },
        draw(ctx) {
            for (var i = 0; i < this.lines_num; i++) {
                let _i = i % this.clrx.length
                ctx.strokeStyle = this.clrx[_i]
                ctx.lineWidth = this.widths[i] || this.line_width
                ctx.beginPath()
                this.draw_spline(ctx, i)
                ctx.stroke()
            }

			// MARKER

			if ( this.markers != null ) {
				ctx.lineWidth = 1.5
				ctx.strokeStyle = 'black'
				document.body.style.cursor = 'auto'
				for (var m of this.markers) {
					this.selected = null
	                this.draw_marker(ctx, m)
	            }
			}
        },
        draw_spline(ctx, i) {
            const layout = this.$props.layout
            const data = this.$props.data
            if (!this.skip_nan) {
                for (var k = 0, n = data.length; k < n; k++) {
                    let p = data[k]
                    let x = layout.t2screen(p[0])
                    let y = layout.$2screen(p[i+1])
                    ctx.lineTo(x, y)
                }
            } else {
                var skip = false
                for (var k = 0, n = data.length; k < n; k++) {
                    let p = data[k]
                    let x = layout.t2screen(p[0])
                    let y = layout.$2screen(p[i+1])
                    if (p[i+1] == null || y !== y) {
                        skip = true
                    } else {
                        if (skip) ctx.moveTo(x, y)
                        ctx.lineTo(x, y)
                        skip = false
                    }
                }
            }
        },

		draw_marker(ctx, p) {
			if (p[1].size == null)
			{
				if (p[1].sel) {
					p[1].size = { height: 20, width: 17, font: 15 };
				} else {
					p[1].size = { height: 14, width: 13, font: 11 };
				}
			}

			if (p[1].position == null) {
				p[1].position = [0, 0]
			}

			if (p[1].direction == null) {
				p[1].direction = "down"
			}

            let layout = this.$props.layout
            let stroke = this.colors.back

            let fill = p[1].color || 'orange'
            let radius = 2
            let height = p[1].size.height
            let width = p[1].size.width
            let x = layout.t2screen(p[0]) - width * 0.5 + p[1].position[0]

            let y = layout.$2screen(p[1].$) + p[1].position[1]
			if (p[1].direction == "down") {
				y = y - (height + height / 5)
			} else {
				y = y + (height + height / 5)
			}

            // Collisions
			if (p[1].direction == "down" &&
					this.mouse.x > x && this.mouse.x < x + width &&
            		this.mouse.y > y && this.mouse.y < y + height
				|| p[1].direction == "up" &&
					this.mouse.x > x && this.mouse.x < x + width &&
	                this.mouse.y < y && this.mouse.y > y - height
			) {
                document.body.style.cursor = 'pointer'
                this.selected = p
                stroke = this.colors.text
            }

            ctx.beginPath()

			if (p[1].direction == "down") {
	            ctx.moveTo(x + radius, y)
	            ctx.lineTo(x + width - radius, y)
	            ctx.quadraticCurveTo(x + width, y, x + width, y + radius)
	            ctx.lineTo(x + width, y + height - radius)
	            ctx.quadraticCurveTo(x + width, y + height, x + width - radius, y + height)
	            ctx.lineTo(x + width * 1 / 2, y + height * 1.2)
	            ctx.lineTo(x + radius, y + height);
	            ctx.quadraticCurveTo(x, y + height, x, y + height - radius)
	            ctx.lineTo(x, y + radius)
	            ctx.quadraticCurveTo(x, y, x + radius, y)
			} else {
				ctx.moveTo(x + radius, y)
	            ctx.lineTo(x + width - radius, y)
	            ctx.quadraticCurveTo(x + width, y, x + width, y - radius)
	            ctx.lineTo(x + width, y - height + radius)
	            ctx.quadraticCurveTo(x + width, y - height, x + width - radius, y - height)
	            ctx.lineTo(x + width * 1 / 2, y - height * 1.2)
	            ctx.lineTo(x + radius, y - height);
	            ctx.quadraticCurveTo(x, y - height, x, y - height + radius)
	            ctx.lineTo(x, y - radius)
	            ctx.quadraticCurveTo(x, y, x + radius, y)
			}

            ctx.lineWidth = 1
            ctx.closePath()
            ctx.fillStyle = fill
            ctx.strokeStyle = stroke
            ctx.fill()
            ctx.stroke()

            ctx.textAlign ='center'
            ctx.fillStyle = p[1].textColor || this.colors.back
            ctx.font = `${ p[1].size.font }px Arial`
            ctx.fillText(p[1].text || '$', x + width/2, (p[1].direction == "down" ? y + height * 0.8 : y - height * 0.3))

		},

        use_for() { return ['Splines', 'DMI'] },
        data_colors() { return this.clrx }
    },
    // Define internal setting & constants here
    computed: {
        sett() {
            return this.$props.settings
        },
        line_width() {
            return this.sett.lineWidth || 0.75
        },
        widths() {
            return this.sett.lineWidths || []
        },
        clrx() {
            let colors = this.sett.colors || []
            let n = this.$props.num
            if (!colors.length) {
                for (var i = 0; i < this.lines_num; i++) {
                    colors.push(this.COLORS[(n + i) % 5])
                }
            }
            return colors
        },
        lines_num() {
            if (!this.$props.data[0]) return 0
            return this.$props.data[0].length - 1
        },
        // Don't connect separate parts if true
        skip_nan() {
            return this.sett.skipNaN
        },
		markers() {
			return this.sett.marker
		}
    },
    data() {
        return {
            COLORS:
            [
                '#42b28a', '#5691ce', '#612ff9',
                '#d50b90', '#ff2316'
            ]
        }
    }
}

</script>
