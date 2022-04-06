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
	const reportId = urlString.match(regex)[0]

	const getReport = async () => {
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
		<h1> Welcome to Dupka! </h1>
		<h2> More Info coming soon </h2>
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
</style>