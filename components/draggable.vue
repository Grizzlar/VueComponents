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
			size: 0,
			offset: null,
			orderMap: {},
			os: null,
		}
	},
	methods: {
		getTMargin(e){
			const s = getComputedStyle(e)
			return (isNaN(parseFloat(s.marginTop)) ? 0 : parseFloat(s.marginTop))
		},
		getVMargin(e){
			const s = getComputedStyle(e)
			return this.getTMargin(e) + (isNaN(parseFloat(s.marginTop)) ? 0 : parseFloat(s.marginTop))
		},
		getNewOrder(event){
			const s = this.dragging.style
			const cOrd = parseInt(this.clone.style.order)
			let mDiff = -1
			let nOrd = -1
			const sxy = this.dragging.getBoundingClientRect()
			let moveDown = true
			Array.from(this.$refs.container.childNodes).forEach(e => {
				if(e.nodeName != '#text' && e != this.clone && e.classList.contains('isClone')){
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
				if(e.nodeName != '#text' && e != this.clone && e.classList.contains('isClone')){
					const ord = parseInt(e.style.order)
					if(moveDown){
						if(nOrd == -2 && ord > cOrd){
							e.style.order = ord-1
							this.orderMap[e.style.order] = this.orderMap[ord]
							setTimeout(() => {
								const clonePos = e.getBoundingClientRect()
								this.orderMap[e.style.order].style.top = (clonePos.y-this.os.y)+'px'
								this.orderMap[e.style.order].style.left = (clonePos.x-this.os.x)+'px'
							}, 50)
							if(ord > MOrd)
								MOrd = ord
						}else if(ord > cOrd && ord <= nOrd){
							e.style.order = ord-1
							this.orderMap[e.style.order] = this.orderMap[ord]
							setTimeout(() => {
								const clonePos = e.getBoundingClientRect()
								this.orderMap[e.style.order].style.top = (clonePos.y-this.os.y)+'px'
								this.orderMap[e.style.order].style.left = (clonePos.x-this.os.x)+'px'
							}, 50)
						}
					}else{
						if(ord < cOrd && ord > nOrd){
							e.style.order = ord + 1
							this.orderMap[e.style.order] = this.orderMap[ord]
							setTimeout(() => {
								const clonePos = e.getBoundingClientRect()
								this.orderMap[e.style.order].style.top = (clonePos.y-this.os.y)+'px'
								this.orderMap[e.style.order].style.left = (clonePos.x-this.os.x)+'px'
							}, 50)
						}
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
			this.clone = dragged
			dragged = this.orderMap[dragged.style.order]
			const s = dragged.style
			s.transition = ''
			const bCRect = dragged.getBoundingClientRect()
			this.offset = {
				x: event.screenX-bCRect.x+this.$refs.container.getBoundingClientRect().x,
				y: event.screenY-bCRect.y+this.$refs.container.getBoundingClientRect().y,
			}
			s.top = (event.screenY-this.offset.y)+'px'
			s.left = (event.screenX-this.offset.x)+'px'
			s.zIndex = 999999
			this.dragging = dragged
			document.addEventListener('mousemove', this.handleDrag)
		},
		drop(event){
			if(event.button != 0) return
			const s = this.dragging.style
			const clonePos = this.clone.getBoundingClientRect()
			s.order = this.clone.style.order
			s.top = (clonePos.y-this.os.y)+'px'
			s.left = (clonePos.x-this.os.x)+'px'
			s.transition = 'top .1s'
			s.zIndex = ""
			document.removeEventListener('mousemove', this.handleDrag)
			this.clone = null
			this.dragging = null
		},
		handleDrag(event){
			const s = this.dragging.style
			s.top = (event.screenY-this.offset.y)+'px'
			s.left = (event.screenX-this.offset.x)+'px'
			this.clone.style.order = this.getNewOrder(event)
			this.orderMap[this.clone.style.order] = this.dragging
		}
	},
	mounted(){
		let i = 0
		this.os = this.$refs.container.getBoundingClientRect()
		Array.from(this.$refs.container.childNodes).forEach(e => {
			if(e.nodeName != '#text'){
				this.orderMap[i] = e
				e.style.cursor = 'pointer'
				e.style.position = 'absolute'
				e.style.transition = 'top .1s'
				const bCRect = e.getBoundingClientRect()
				const clone = document.createElement('div')
				const size = bCRect.height+this.getVMargin(e)
			  clone.classList.add('isClone')
				clone.style.opacity = 0;
				clone.style.height = size+'px'
				clone.style.order = i++
				this.$refs.container.appendChild(clone)
				setTimeout(() => {
					const clonePos = clone.getBoundingClientRect()
					e.style.top = (clonePos.y-this.os.y)+'px'
					e.style.left = (clonePos.x-this.os.x)+'px'
				}, 50)
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
