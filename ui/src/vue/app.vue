

<script>
    import Vue from 'vue'

    var Router = VueRouter.Router;
    var Route = VueRouter.Route;
    var Link = VueRouter.Link;
    var browserHistory = VueRouter.browserHistory;

    function getParent(path) {
        let paths = path.split("/")
        return (paths.length >= 2) ? "/" + _.join(_.take(paths,paths.length-1),"/") : "/"
    }

    const App = {
        render() {
            <div>
				{this.props.children}
			</div>
        }
    }

    const Player = {
        render() {
            let path = this.props.params.splat;
            return (
                <div className="player" key={path}>
                    <div className="stage">
                        <video
                            className="video-js vjs-default-skin vjs-16-9 vjs-big-play-centered"
                            ref={(c) => this._video = c}
                            width="100%" controls >
                            <source
                                src={"/playlist/720/" + path} 
                                type="application/x-mpegURL" />
                        </video>
                    </div>
                    <Link to={getParent(path)} className="back">
                        <span className="glyphicon glyphicon-chevron-left" aria-hidden="true"></span>
                    </Link>
                </div>
            )
        },
        
        mounted() {
            this.video = ReactDOM.findDOMNode(this._video);
            $(this.video).attr('x-webkit-airplay','allow');
            $(this.video).attr('airplay','allow');
            this.player = videojs(this.video,{
                //plugins: { airplayButton: {} }
            });
            this.player.play();
        }

        //Chua co componentWillUnmount()
    }

    const Folder = {
        render() {
            <Link to={"/list/" + this.props.path }  >
				<div className="list-item folder" key={this.props.path}>
					<div className="left">
						<div className="frame">
							<div className="inner">
								<span className="glyphicon glyphicon-folder-open" aria-hidden="true"></span>
							</div>
						</div>
					</div>
					<div className="right">
						{this.props.name}
					</div>
				</div>
			</Link>
        }
    }

    const Loader = {
        render() {
            <div className="loader">
				<img width="30" height="30" src="/ui/assets/img/loader.svg" />
			</div>
        }
    }

    const EmptyMessage = {
        render() {
            <div className="empty-message">
				<p>No folders or videos found in folder :-(</p>
			</div>
        }
    }

    const Duration = (props) => {
        function pad(str) {
            var pad = "00"
            return pad.substring(0, pad.length - str.length) + str
        }
        let time = parseInt(props.duration)
        let hours = Math.floor(time / 3600)
        let minutes = Math.floor((time - hours * 3600) / 60)
        let seconds = (time - hours * 3600 - minutes * 60)
        return (
            <span>
                {pad(hours)}h{pad(minutes)}m{pad(seconds)}s
            </span>
        );
    }

    const Video = {
        render() {
            let time = Math.min(30.0,Math.ceil(this.props.info.duration * 0.1))
            return (
                <Link to={"/play/" + this.props.path} >
                    <div className="list-item video" key={this.props.path}>
                        <div className="left">
                            <div className="frame" style={{"backgroundImage": "url('/frame/" + this.props.path+"?t="+time+"')"}} >
                                <div className="inner">
                                    <span className="glyphicon glyphicon-play-circle" aria-hidden="true"></span>
                                </div>
                            </div>
                        </div>
                        <div className="right">
                            <p>
                                {this.props.name}&nbsp;
                                (<a onClick={(e) => {e.preventDefault(); window.location = "/download/" + this.props.path}} target="_blank">Download</a>)
                            </p>
                            <p className="video-info">
                                <span className="glyphicon glyphicon-time"/> <Duration duration={this.props.info.duration} />
                                &nbsp;| {moment(this.props.info.lastModified).format("MMM DD YYYY, hh:mm")}
                            </p>
                        </div>
                    </div>
                </Link>
            )
        }
    }

    const List = {
        data:() => ({
            'videos': null,
			'folders': null
        }),

        fetch(path) {
            this.setData({
                'folders': null,
			    'videos': null
            }),
            $.get('/list/' + path,(data) => {
                this.setData({
                    'folders': data.folders,
                    'videos': data.videos
                })
            });
        },

        render () {
            let loader = (!this.state.folders) ? <Loader/> : null;

            let folders = []
            let videos = []
            if (this.state.folders) {
                folders = this.state.folders.map((folder) => <Folder key={folder.name} name={folder.name} path={folder.path} />)
                videos = this.state.videos.map((video) => <Video name={video.name} info={video.info} path={video.path} key={video.name} />)
            }
            let empty = (this.state.folders != null && (videos.length + folders.length) == 0) ? <EmptyMessage/> : null
            return (
                <div className="list">
                    <header className="header">
                        <div className="header-content">
                            <div className="header-left">
                                <Link to={getParent(this.props.params.splat)}>
                                    <span className="glyphicon glyphicon-circle-arrow-left"/>
                                </Link>
                            </div>
                            <div className="header-center">
                                {"/" + this.props.params.splat || "/"}
                            </div>
                            <div className="header-right">

                            </div>
                        </div>
                    </header>
                    <div className="">
                        <div className="list-items">
                            {loader}
                            {folders}
                            {videos}
                            {empty}
                        </div>
                    </div>
                </div>
            )
        },

        mounted() {
            var path = this.props.params.splat || "";
		    this.fetch(path)
        }

        // Chua co compnemtWillReceiveProps()
    }

    videojs.Hls.xhr.beforeRequest = function(options) {
        options.timeout = 30000;
        return options;
    }

    const h = VueRouter.useRouterHistory(History.createHistory)({
        basename: '/ui'
    })

    new Vue({
        el: document.getElementById('app'),
        render() {
            return (<Router history={h} >
                        <Route component={App}>
                            <Route name="list" path="list/*"  component={List} />
                            <Route name="play" path="play/*"  component={Player} />
                            <Route path="*" component={List}/>
                        </Route>
                    </Router>)
        }
    })
</script>


