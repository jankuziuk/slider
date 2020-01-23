import React from 'react';
import './XSlider.css';

class XSlider extends React.Component {
    constructor(props) {
        super(props);

        this.params = {
            selected: 2,
            ...props,
        };

        this.sliderRef = React.createRef();

        this.state = {
            activeSlide: this.params.selected,
            lastActiveSlide: this.params.selected,
            isDruging: false,
        }
    }

    componentDidMount () {
        this.setTranslate();

    }

    setTranslate() {
        if (this.sliderRef.current) {
            this.setState({
                lastActiveSlide: this.state.activeSlide,
                translate: this.getTranslate(this.state.activeSlide),
                lastTranslate: this.getTranslate(this.state.activeSlide),
                transition: 'none'
            });
        }
    }

    getTranslate(index = 0, offset = 0) {
        if (this.sliderRef.current) {
            return this.sliderRef.current.clientWidth * index - offset
        }
    }

    getDragX(event, isTouch = true) {
        console.log(event.screenX);
        return isTouch ?
            event.touches[0].pageX :
            event.screenX;
    }

    dragStart = (event) => {
        console.log(event.type);
        this.setState({ 
            isDruging: true,
            dragStart: this.getDragX(event, event.type === 'touchstart'),
            transition: 'none'
        })
    }

    dragMove = (event) => {
        if (this.state.isDruging) {
            const x = this.getDragX(event, event.type === 'touchmove');
            const offset = this.state.dragStart - x;
            const statusTranslate = this.state.lastTranslate - this.state.translate;
            let newIndex = this.state.lastActiveSlide;

            console.log(statusTranslate);
            if (Math.abs(statusTranslate) >= Math.ceil(this.sliderRef.current.clientWidth * 0.1)) {
                newIndex = statusTranslate > 0 ? this.state.lastActiveSlide - 1 : this.state.lastActiveSlide + 1;
            }
            
            this.setState({
                dragStart: x,
                activeSlide: newIndex,
                translate: this.state.translate + offset,
            })
        }
    }

    dragEnd = (event) => {
        this.setState({ 
            isDruging: false,
            getDragX: 0,
            transition: 'transform 200ms'
        });
        this.goToSlide(this.state.activeSlide);
    }

    goToSlide = (index) => {
        this.setState({
            activeSlide: index,
            lastTranslate: this.getTranslate(index),
            translate: this.getTranslate(index),
            transition: 'transform 200ms'
        })
    }

    prev = () => {
        this.setState({
            activeSlide: this.state.activeSlide - 1,
            lastTranslate: this.getTranslate(this.state.activeSlide - 1),
            translate: this.getTranslate(this.state.activeSlide - 1),
            transition: 'transform 200ms'
        })
    }

    next = () => {
        this.setState({
            activeSlide: this.state.activeSlide + 1,
            lastTranslate: this.getTranslate(this.state.activeSlide + 1),
            translate: this.getTranslate(this.state.activeSlide + 1),
            transition: 'transform 200ms'
        })
    }

    render() {
        return (
            <div className='xslider' ref={this.sliderRef}>
                <div className='xslider-items'
                    style={{
                        'transform': `translateX(-${this.state.translate}px)`,
                        'transition': this.state.transition
                    }}
                    onMouseDown={this.dragStart}
                    onTouchStart={this.dragStart}
                    onMouseMove={this.dragMove}
                    onTouchMove={this.dragMove}
                    onMouseUp={this.dragEnd}
                    onTouchEnd={this.dragEnd}
                >
                    {this.props.children}
                </div>
                <button onClick={this.prev}>prev</button>
                <button onClick={this.next}>next</button>
            </div>
        );
    }
}

export default XSlider;
