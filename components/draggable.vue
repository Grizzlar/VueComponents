<template>
	<div ref="container" @mousedown="drag" @mouseup="drop">
    <slot></slot>	
	</div>
</template>
<script>
export default {
	data(){
		return {
			dragging: null,
			clone: null,
			offset: null
		}
	},
	methods: {
		getVMargin(e){
			const s = getComputedStyle(e)
			return (isNaN(parseFloat(s.marginTop)) ? 0 : parseFloat(s.marginTop)) + (isNaN(parseFloat(s.marginTop)) ? 0 : parseFloat(s.marginTop))
		},
		getNewOrder(event){
			const s = this.dragging.style
			const cOrd = parseInt(this.clone.style.order)
			let mDiff = -1
			let nOrd = -1
			const sxy = this.dragging.getBoundingClientRect()
			let moveDown = true
			Array.from(this.$refs.container.childNodes).forEach(e => {
				if(e.nodeName != '#text' && e != this.dragging && !e.classList.contains('isClone')){
					let { x, y } = e.getBoundingClientRect()
					let diff = y-sxy.y
					if(diff > 0 && (mDiff == -1 || diff < mDiff)){
						mDiff = diff
						nOrd = e.style.order
					}
				}
			})
			nOrd--
			if(nOrd == cOrd)
				return cOrd
			if(nOrd < cOrd && nOrd >= -1){
				moveDown = false
			}
			let MOrd = 0
			Array.from(this.$refs.container.childNodes).forEach(e => {
				if(e.nodeName != '#text' && e != this.dragging && !e.classList.contains('isClone')){
					const ord = parseInt(e.style.order)
					if(moveDown){
						if(nOrd == -2 && ord > cOrd){
							e.style.order = ord-1
							if(ord > MOrd)
								MOrd = ord
						}else if(ord > cOrd && ord <= nOrd)
							e.style.order = ord-1
					}else{
						if(ord < cOrd && ord > nOrd)
							e.style.order = ord + 1
					}
				}
			})
			if(!moveDown) nOrd++
			if(nOrd == -2)
				return MOrd == 0 ? cOrd : MOrd
			else
				return nOrd
		},
		drag(event){
			if(event.button != 0) return
			let dragged = event.srcElement
			while(dragged.parentNode !== this.$refs.container){
				dragged = dragged.parentNode
			}
			const s = dragged.style
			const bCRect = dragged.getBoundingClientRect()
			this.offset = {
				x: event.screenX-bCRect.x+this.$refs.container.getBoundingClientRect().x,
				y: event.screenY-bCRect.y+this.$refs.container.getBoundingClientRect().y,
			}
			this.clone = document.createElement('div')
			this.clone.classList.add('isClone')
			this.clone.style.opacity = 0;
			this.clone.style.height = bCRect.height+this.getVMargin(dragged)+'px'
			this.clone.style.order = dragged.style.order
			this.$refs.container.appendChild(this.clone)
			s.top = (event.screenY-this.offset.y)+'px'
			s.left = (event.screenX-this.offset.x)+'px'
			s.position = "absolute"
			s.zIndex = 999999
			this.dragging = dragged
			document.addEventListener('mousemove', this.handleDrag)
		},
		drop(event){
			if(event.button != 0) return
			const s = this.dragging.style
			s.order = this.clone.style.order
			this.$refs.container.removeChild(this.clone)
			document.removeEventListener('mousemove', this.handleDrag)
			s.zIndex = ""
			s.position = ""
			s.top = ""
			s.left = ""
			this.dragging = null
		},
		handleDrag(event){
			const s = this.dragging.style
			s.top = (event.screenY-this.offset.y)+'px'
			s.left = (event.screenX-this.offset.x)+'px'
			this.clone.style.order = this.getNewOrder(event)
		}
	},
	mounted(){
		Array.from(this.$refs.container.childNodes).forEach((e, i) => {
			if(e.nodeName != '#text'){
				e.style.order = i
				e.style.cursor = 'pointer'
			}
		})
	}
}
</script>
<style scoped>
	div{
		display: flex; flex-direction: column;
	}
</style>
