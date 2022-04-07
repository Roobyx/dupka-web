<script lang='ts' setup>
	import { initializeApp } from 'firebase/app';
	import { getFirestore, doc, getDoc } from 'firebase/firestore/lite';
	import { getAuth, signInWithEmailAndPassword } from "firebase/auth";
	import { reactive } from 'vue'

	interface IReport {
		address: string
		timestamp: {
			seconds: number
		},
		rating: number,
		rates: string[],
		location: {
			coords: {
				latitude: number,
				longitude: number
			}
		}
		reportImage: string,
	}

	interface ILocation {
		lat: number,
		lng: number
	}

	interface IStore {
		report: IReport,
		center: ILocation,
		markers: {
			position: ILocation
		}[],
		reportFetched: boolean,
		setReport: (value: IReport) => void,
		setReportReady: () => void,
		created: () => void
	}

	const firebaseConfig = {
		databaseURL: import.meta.env.VITE_FBC_dbUrl,
		apiKey: import.meta.env.VITE_FBC_apiKey,
		authDomain: import.meta.env.VITE_FBC_authDomain,
		projectId: import.meta.env.VITE_FBC_projectId,
		storageBucket: import.meta.env.VITE_FBC_storageBucket,
		messagingSenderId: import.meta.env.VITE_FBC_messagingSenderId,
		appId: import.meta.env.VITE_FBC_appId,
		measurementId: import.meta.env.VITE_FBC_measurementId
	}

	const logParam = {
		name: import.meta.env.VITE_CRED_fName,
		ps: import.meta.env.VITE_CRED_fp,
		mtk: import.meta.env.VITE_CRED_mTk,
		gk: import.meta.env.VITE_CRED_gK
	}

	const store: IStore = reactive({
		report: {
			address: '',
			timestamp: {
				seconds: 0
			},
			rating: 0,
			rates: [],
			reportImage: '',
			location: {
				coords: {
					latitude: 0,
					longitude: 0,
				}
			}
		},
		center: {
			lat: 51.093048, 
			lng: 6.842120
		},
		markers: [
			{
				position: {
					lat: 51.093048,
					lng: 6.842120
				},
			}
		],
		reportFetched: false,
		setReport(value: IReport) {
			this.report = value
		},
		setReportReady() {
			this.reportFetched = true
		},
		created() {
			//@ts-ignore
			this.mapbox = Mapbox;
		}
	})
	
	const getTime = (seconds: number) => {
		const time = new Date(Date.UTC(1970, 0, 1))
		time.setSeconds(seconds)
		const monthString = new Intl.DateTimeFormat('bg-BG', {month: 'long'}).format(time)
		const timeString = time.getDate() + ' ' + monthString  + ' ' + time.getHours() + ':' + time.getMinutes()
		return timeString
	}

	const app = initializeApp(firebaseConfig)
	const db = getFirestore(app)

	const currentUrl = window.location
	const urlString = currentUrl.toString()
	const regex = /(?<=r:)[\s\S]*/g;
	const idMatch = urlString.match(regex)
	const reportId = idMatch ? idMatch[0] : null

	const getReport = async () => {
		if (reportId) {
			try {
				const res = await signInWithEmailAndPassword(getAuth(), logParam.name, logParam.ps)
				const user = JSON.parse(JSON.stringify(res.user))

				if( user ) {
					// console.log('successfully logged user: ', res.user.email)

					// Get a list of cities from your database
					const docRef = doc(db, "reports", reportId);
					const docSnap = await getDoc(docRef);

					if (docSnap.exists()) {
						// @ts-ignore
						store.setReport(docSnap.data())
						store.setReportReady()
						// console.log('report exists')
					} else {
						console.log("No such document!");
					}
				}
			} catch (e: any) {
					console.log('Got error while logging: ', e.message)
			}
		}
	}

	getReport()
</script>

<template>
	<link
	href="https://api.tiles.mapbox.com/mapbox-gl-js/v0.53.0/mapbox-gl.css"
	rel="stylesheet"
	/>

	<div class='report-container'
		v-if="store.reportFetched"
	>
		<picture class="photo-container">
			<img class='report-image' :src="store.report.reportImage" alt="">
		</picture>

		<div class="report-info">
			<h3 class="report-address"> {{ store.report.address }} </h3>

			<div class="info report-time"> 
				<div class="icon icon-time">
					<svg xmlns="http://www.w3.org/2000/svg" class="ionicon" viewBox="0 0 512 512"><title>Time</title><path d="M256 48C141.13 48 48 141.13 48 256s93.13 208 208 208 208-93.13 208-208S370.87 48 256 48zm96 240h-96a16 16 0 01-16-16V128a16 16 0 0132 0v128h80a16 16 0 010 32z"/></svg>
				</div>
				<div class="time-data">
					{{ getTime(store.report.timestamp.seconds) }}
				</div>
			</div>
			
			<div class="info report-rating"> 
				<div class="icon icon-star">
					<svg xmlns="http://www.w3.org/2000/svg" class="ionicon" viewBox="0 0 512 512"><path d="M394 480a16 16 0 01-9.39-3L256 383.76 127.39 477a16 16 0 01-24.55-18.08L153 310.35 23 221.2a16 16 0 019-29.2h160.38l48.4-148.95a16 16 0 0130.44 0l48.4 149H480a16 16 0 019.05 29.2L359 310.35l50.13 148.53A16 16 0 01394 480z"/></svg>
				</div>
				<div class="reviews-rating">
					<!-- <div class="reviews-digit">{{ store.report.rating }}</div> -->
					<div class="reviews-count"> 
						{{ store.report.rating }}
						out of 
						<span class="hightlight">{{ store.report.rates.length }} </span>
						ratings
					</div>
				</div>
			</div>
			<div class="share-icons">
				<h4>Share on:</h4>
				<div class="share icon icon-direction">
					<svg xmlns="http://www.w3.org/2000/svg" class="ionicon" viewBox="0 0 512 512"><title>Compass</title><circle cx="256" cy="256" r="24"/><path d="M256 48C141.31 48 48 141.31 48 256s93.31 208 208 208 208-93.31 208-208S370.69 48 256 48zm105.07 113.33l-46.88 117.2a64 64 0 01-35.66 35.66l-117.2 46.88a8 8 0 01-10.4-10.4l46.88-117.2a64 64 0 0135.66-35.66l117.2-46.88a8 8 0 0110.4 10.4z"/></svg>
					<a target='_blank' :href="'https://www.google.com/maps/search/?api=1&query=' + store.report.location.coords.latitude + ',' + store.report.location.coords.longitude"> Find in Maps </a>
				</div>
				<!-- <div class="share icon icon-mail">
					<svg xmlns="http://www.w3.org/2000/svg" class="ionicon" viewBox="0 0 512 512"><title>Mail</title><path d="M424 80H88a56.06 56.06 0 00-56 56v240a56.06 56.06 0 0056 56h336a56.06 56.06 0 0056-56V136a56.06 56.06 0 00-56-56zm-14.18 92.63l-144 112a16 16 0 01-19.64 0l-144-112a16 16 0 1119.64-25.26L256 251.73l134.18-104.36a16 16 0 0119.64 25.26z"/></svg>
				</div> -->
				<!-- <div class="share icon icon-link">
					<svg xmlns="http://www.w3.org/2000/svg" class="ionicon" viewBox="0 0 512 512"><title>Copy</title><path d="M408 480H184a72 72 0 01-72-72V184a72 72 0 0172-72h224a72 72 0 0172 72v224a72 72 0 01-72 72z"/><path d="M160 80h235.88A72.12 72.12 0 00328 32H104a72 72 0 00-72 72v224a72.12 72.12 0 0048 67.88V160a80 80 0 0180-80z"/></svg>
					<span> {{ urlString }} </span>
				</div> -->
				<div class="share icon icon-twitter">
					<svg xmlns="http://www.w3.org/2000/svg" class="ionicon" viewBox="0 0 512 512"><title>Logo Twitter</title><path d="M496 109.5a201.8 201.8 0 01-56.55 15.3 97.51 97.51 0 0043.33-53.6 197.74 197.74 0 01-62.56 23.5A99.14 99.14 0 00348.31 64c-54.42 0-98.46 43.4-98.46 96.9a93.21 93.21 0 002.54 22.1 280.7 280.7 0 01-203-101.3A95.69 95.69 0 0036 130.4c0 33.6 17.53 63.3 44 80.7A97.5 97.5 0 0135.22 199v1.2c0 47 34 86.1 79 95a100.76 100.76 0 01-25.94 3.4 94.38 94.38 0 01-18.51-1.8c12.51 38.5 48.92 66.5 92.05 67.3A199.59 199.59 0 0139.5 405.6a203 203 0 01-23.5-1.4A278.68 278.68 0 00166.74 448c181.36 0 280.44-147.7 280.44-275.8 0-4.2-.11-8.4-.31-12.5A198.48 198.48 0 00496 109.5z"/></svg>
					<a target="_blank" :href="'https://twitter.com/intent/tweet?text=' + urlString"> Tweet it </a>
				</div>
			</div>
		</div>

		<div class="map-container">
			<iframe :src="'https://www.google.com/maps/embed/v1/place?key=' + logParam.gk + '&q='+ store.report.location.coords.latitude + ',' + store.report.location.coords.longitude" loading="lazy" style="border:0px"></iframe>
		</div>


	</div>

	<div class='index'
		v-if="!store.reportFetched"
	>
		<!-- <h1> Welcome to Dupka! </h1>
		<hr>
		<div>
			<div>
				<h2>About:</h2>
			</div>
			<p>
				The idea of the app is to allow you to report immidiate infrastructure problems around you and in that way build an interactive map of the state of the country's infrastructural problems.
			</p>
			<hr>
			<p>
				The project started as a course work, but now I'd like to see where I can push its development to.
			</p>
		</div> -->

		<nav class="accordion arrows">
			<header class="box">
				<label for="acc-close" class="box-title">Dupka app</label>
			</header>

			<input type="radio" name="accordion" id="cb1" />
			<section class="box">
				<label class="box-title" for="cb1">About</label>
				<label class="box-close" for="acc-close"></label>
				<div class="box-content">
					<p>
						The idea of the app is to allow you to report immidiate infrastructure problems around you and in that way build an interactive map of the state of the country's infrastructural problems.
					</p>
					<hr>
					<p>
						The project started as a course work, but now I'd like to see where I can push its development to.
					</p>
				</div>
			</section>

			<input type="radio" name="accordion" id="cb2" />
			<section class="box">
				<label class="box-title" for="cb2">Download & install</label>
				<label class="box-close" for="acc-close"></label>
				<div class="box-content">
					<p>
						The project currently consists of 2 parts - Mobile app and Web viewer.
					</p>
					<div>
						<h3> To install the Android app </h3>
					</div>

					<ol>
						<li>
							From your phone, download the .apk from:  
							<a href="https://u.pcloud.link/publink/show?code=XZTga5VZCdm0LE5BiFhHOmkoVrH6Yh1UpdXy"> here </a>.
						</li>
						<li>Click "Direct download" (the lighter button of the 2 below the icon)</li>
						<li>Open the file and it will prompt for installation. (It may be needed to allow "Installing from unknown sources"</li>
					</ol>
				</div>
			</section>

			<input type="radio" name="accordion" id="cb3" />
			<section class="box">
				<label class="box-title" for="cb3">Usage</label>
				<label class="box-close" for="acc-close"></label>
				<div class="box-content">
					<p>
						<!-- The app is designed to use minimal personal information such as personal details or location data.
						<br> -->
						You can choose to log in anonymously - this bypasses the account creation.
						<br>
						or
						<br>
						you can register an account with username and password
						<br>
						<br>
						<b>Note: <i>Currently there is no email verification or rest, so remember your password</i></b>
					</p>
					<p>
						When you log in, you can scroll trough reports or use the <b>+</b> icon to start a new report.
					</p>
					<div>
						<h3> Reporting </h3>
					</div>
					<div>
						<ol>
							<li>
								When you spot something you want to report, like a hole in the road. Click the <b>+</b> icon, this will open your camera, take a photo and click "Save" if you like it.
							</li>
							<li>
								Choose to take a photo or upload from files.
							</li>
							<li>
								On the next page, just "rate" the severity of the problem you just took a photo of and press "Submit report".
							</li>
							<li>
								You have created a report. You will be able to see it in the feed, onces its approved by the admin.
							</li>
							<li>
								<b>
									For admin access during the tests, please contact me. :)
								</b>
							</li>
						</ol>
					</div>

				</div>
			</section>

			<input type="radio" name="accordion" id="cb4" />
			<section class="box">
				<label class="box-title" for="cb4">BUGS!</label>
				<label class="box-close" for="acc-close"></label>
				<div class="box-content">
					<p>
						Please report bugs to: <a href="https://github.com/Roobyx/dupka">https://github.com/Roobyx/dupka</a>
					</p>
					<hr>
					<div><h3>Currently known problems:</h3></div>
					<ul>
						<li>The items in the feed appear in random order</li>
						<li>On rare occasions, the app crashes when clicking "+" icon for report</li>
						<li>Profile -> "Logout" doesnt work at the moment</li>
					</ul>
				</div>
			</section>

			<input type="radio" name="accordion" id="acc-close" />
		</nav>
	</div>

</template>

<style scoped>
	.report-container {
		display: grid;
		grid-template-rows: 2fr 2fr;
		grid-template-columns: 3fr 1fr;
		max-width: 100%;
		width: 100%;
		position: relative;
		align-items: center;
		justify-content: center;
		height: 100vh;
	}
	.map-container {
		height: 100%;
		
		grid-row: 2/2;
		grid-column: 2/2;
	}
	.map-container iframe {
		height: 100%;
		width: 100%;
		box-shadow: 0 1px 2px rgb(60 64 67 / 30%), 0 2px 6px 2px rgb(60 64 67 / 15%);
	}
	.vue-map {
		overflow: auto;
	}
	.photo-container {
		display: grid;
		height: 100%;
		grid-row: 1/3;

		border-right: 1px solid #e8eaed;

	}
	.photo-wrapper {
		max-width: 100%;
		max-height: 100vh;
		margin: auto;
	}
	.report-image {
		max-width: 100%;
		max-height: 80vh;
		margin: auto;
	}
	.report-info {
		padding: 30% 10%;
		height: 100%;
		border-bottom: 1px solid #e8eaed;
		box-shadow: 0 1px 2px rgb(60 64 67 / 30%), 0 2px 6px 2px rgb(60 64 67 / 15%);
	}
	.report-address {
		font-family: Roboto, Arial,sans-serif;
		font-size: 1.375rem;
		font-weight: 400;
		letter-spacing: 0;
		line-height: 1.75rem;
	}
	.reviews-digit {
		font-weight: 400;
	}
	.reviews-count {
		border: 1px solid transparent;
		align-items: center;
		text-align: center;
	}
	.hightlight {
		color: #1a73e8;
	}
	.icon svg {
		width: 18px;
		height: 18px;
		margin-right: 10px;
	}
	.info {
		display: flex;
		margin: 10px 0;
		padding: 10px 0;
		border-bottom: 1px solid #e8eaed;
	}
	.share {
		display: flex;
	}
	.share-icons {
		display: flex;
		flex-direction: column;
		gap: 10px;
	}

	.icon-twitter {
		fill: #1DA1F2;
	}
	.icon-link {
		fill: #171717;
	}
	.icon-direction {
		fill: #FF5A5F;
	}
	.icon-mail {
		fill: #00A699;
	}
	.icon-time {
		fill: #E60023;
	}
	.icon-star {
		fill:#fbbc04;
	}
	.index {
		display: flex;
		height: 100vh;
		justify-content: center;
		text-align: center;
		flex-direction: column;
	}
	@media screen and (max-width: 991px) {
		.map-container {
			display: none;
		}
		.report-container {
			height: auto;
			display: block;
			padding: 10px;
		}
		.report-info {
			padding: 10px;
			box-shadow: none;
		}
	}


	.accordion {
		margin: auto;
		width: 400px;
		text-align: left;
	}
	.accordion input {
		display: none;
	}
	.box {
		position: relative;
		background: white;
		height: 64px;
		transition: all .15s ease-in-out;
	}
	.box::before {
		content: '';
		position: absolute;
		display: block;
		top: 0;
		bottom: 0;
		left: 0;
		right: 0;
		pointer-events: none;
		box-shadow: 0 -1px 0 #e5e5e5,0 0 2px rgba(0,0,0,.12),0 2px 4px rgba(0,0,0,.24);
	}
	header.box {
		background: #00BCD4;
		z-index: 100;
		cursor: initial;
		box-shadow: 0 -1px 0 #e5e5e5,0 0 2px -2px rgba(0,0,0,.12),0 2px 4px -4px rgba(0,0,0,.24);
	}
	header .box-title {
		margin: 0;
		font-weight: normal;
		font-size: 16pt;
		color: white;
		cursor: initial;
	}
	.box-title {
		width: calc(100% - 40px);
		height: 64px;
		line-height: 64px;
		padding: 0 20px;
		display: inline-block;
		cursor: pointer;
		-webkit-touch-callout: none;-webkit-user-select: none;-khtml-user-select: none;-moz-user-select: none;-ms-user-select: none;user-select: none;
	}
	.box-content {
		width: calc(100% - 40px);
		padding: 30px 20px;
		font-size: 11pt;
		color: rgba(0,0,0,.54);
		display: none;
	}
	.box-close {
		position: absolute;
		height: 64px;
		width: 100%;
		top: 0;
		left: 0;
		cursor: pointer;
		display: none;
	}
	input:checked + .box {
		height: auto;
		margin: 16px 0;
		box-shadow: 0 0 6px rgba(0,0,0,.16),0 6px 12px rgba(0,0,0,.32);
	}
	input:checked + .box .box-title {
		border-bottom: 1px solid rgba(0,0,0,.18);
	}
	input:checked + .box .box-content,
	input:checked + .box .box-close {
		display: inline-block;
	}
	.arrows section .box-title {
		padding-left: 44px;
		width: calc(100% - 64px);
	}
	.arrows section .box-title:before {
		position: absolute;
		display: block;
		content: '\203a';
		font-size: 18pt;
		left: 20px;
		top: -2px;
		transition: transform .15s ease-in-out;
		color: rgba(0,0,0,.54);
	}
	input:checked + section.box .box-title:before {
		transform: rotate(90deg);
	}

	p {
		margin: 10px 0;
	}

	ol {
		padding-left: 20px;
	}

	ul {
		padding-left: 20px;
	}

	ol li {
		margin: 10px 0;
	}

	ul li {
		margin: 10px 0;
	}

	hr {
		margin: 4px 0;
	}

</style>