<script>
// Segment renderer.

import Overlay from '../../mixins/overlay.js'

export default {
    name: 'Segment',
    mixins: [Overlay],
    methods: {
        meta_info() {
            return { author: 'C451', version: '1.0.0' }
        },
        draw(ctx) {

            if (!this.p1 || !this.p2) return

            ctx.lineWidth = this.line_width
            ctx.strokeStyle = this.color
            ctx.beginPath()

            const layout = this.$props.layout

			let x1 = layout.t2screen(this.p1[0])
			let y1 = layout.$2screen(this.p1[1])
			if (this.div_segment) {
				if (this.p1[1] < this.p2[1]) {
					y1 -= this.line_width
				} else {
					y1 += this.line_width
				}
			}
			ctx.moveTo(x1, y1)

			let x2 = layout.t2screen(this.p2[0])
			let y2 = layout.$2screen(this.p2[1])
			if (this.div_segment) {
				if (this.p1[1] < this.p2[1]) {
					y2 -= this.line_width
				} else {
					y2 += this.line_width
				}
			}
            ctx.lineTo(x2, y2)

            ctx.stroke()

			if (this.div_segment) {
				ctx.fillStyle = this.color
				ctx.beginPath();
				if (this.p1[1] < this.p2[1]) {
					ctx.moveTo(x1 - this.line_width, y1 - Math.round(this.line_width / 2))
					ctx.lineTo(x1, y1 + Math.round(this.line_width * 1.5));
					ctx.lineTo(x1 + this.line_width, y1 - Math.round(this.line_width / 2));
				} else {
					ctx.moveTo(x1 - this.line_width, y1 + Math.round(this.line_width / 2))
					ctx.lineTo(x1, y1 - Math.round(this.line_width * 1.5));
					ctx.lineTo(x1 + this.line_width, y1 + Math.round(this.line_width / 2));
				}
				ctx.fill();

				ctx.beginPath();
				if (this.p1[1] < this.p2[1]) {
					ctx.moveTo(x2 - this.line_width, y2 - Math.round(this.line_width / 2))
					ctx.lineTo(x2, y2 + Math.round(this.line_width * 1.5));
					ctx.lineTo(x2 + this.line_width, y2 - Math.round(this.line_width / 2));
				} else {
					ctx.moveTo(x2 - this.line_width, y2 + Math.round(this.line_width / 2))
					ctx.lineTo(x2, y2 - Math.round(this.line_width * 1.5));
					ctx.lineTo(x2 + this.line_width, y2 + Math.round(this.line_width / 2));
				}
				ctx.fill();
			}
        },
        use_for() { return ['Segment'] },
        data_colors() { return [this.color] }
    },
    // Define internal setting & constants here
    computed: {
        sett() {
            return this.$props.settings
        },
        p1() {
            return this.$props.settings.p1
        },
        p2() {
            return this.$props.settings.p2
        },
        line_width() {
            return this.sett.lineWidth || 0.9
        },
		div_segment() {
            return this.sett.divergenceSegment
        },
        color() {
            const n = this.$props.num % 5
            return this.sett.color || this.COLORS[n]
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
