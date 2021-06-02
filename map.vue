<template>
	<div class="map-chart-container">
		<v-chart
			:forceFit="true"
			:height="708"
			:data="geoData"
			:padding="[0, 50, 10]"
		>
			<v-tooltip
				:showTitle="false"
				:useHtml="true"
				:htmlContent="htmlContentHandle"
			/>
			<v-view :data="data">
				<v-polygon
					:position="polygonOpts.position"
					:label="polygonOpts.label"
					:color="polygonOpts.color"
				/>
			</v-view>
		</v-chart>
		<div id="map"></div>
	</div>
</template>

<script>
import * as _ from 'lodash';
const DataSet = require('@antv/data-set');
const colors = [
	'#3366cc',
	'#dc3912',
	'#ff9900',
	'#109618',
	'#990099',
	'#0099c6',
	'#dd4477',
	'#66aa00',
	'#b82e2e',
	'#316395',
	'#994499',
	'#22aa99',
	'#aaaa11',
	'#6633cc',
	'#e67300',
	'#8b0707',
	'#651067',
	'#329262',
	'#5574a6',
	'#3b3eac',
];
// 地图中的图例配置
const polygonOpts = {
	position: 'longitude*lantitude',
	label: [
		'name',
		{
			textStyle: {
				fill: '#fff',
				fontSize: 12,
			},
		},
	],
	style: [
		'name',
		{
			textStyle: {
				fill: '#fff',
				fontSize: 10,
				shadowBlur: 2,
				shadowColor: 'rgba(0, 0, 0, .45)',
			},
		},
	],
	color: ['value', '#BAE7FF-#1890FF-#0050B3'],
};
export default {
	props: {
		// 用户数据列表，自定义布局时不需要考虑key，使用contentTpl和itemTpl时需要对应name和value（具体未了解）
		// [{
		// 	name: '',
		// 	key: '',
		// 	value: '',
		// 	percent: '',
		// }]
		list: {
			default: () => [],
		},
	},
	data() {
		return {
			data: [],
			geoData: {},
			polygonOpts,
			name: '',
			currentAreaNode: null,
			districtExplorer: null,
			mapData: null
		};
	},
	methods: {
		// 自定义tooltip的浮层样式，useHtml为true才会生效
		htmlContentHandle(title, item) {
			let data = item[0].point.point;
			return `<div class="g2-tooltip">
				<div class="g2-tooltip-list">
					<b>${data.name}</b>
					<b>${data.value}万元</b>
					<b>${data.percent}%</b>
					<p>占比</p>
				</div>
			</div>`;
		},
		//加载区域
		loadAreaNode(adcode, callback) {
			this.districtExplorer.loadAreaNode(adcode, (error, areaNode) => {
				if (error) {
					if (callback) {
						callback(error);
					}
					return;
				}

				const adcode = areaNode.getAdcode();
				const geoJSON = areaNode.getSubFeatures(); // 生成地图所需的地理位置坐标数据
				const name = areaNode.getName();
				if (
					!geoJSON ||
					(this.currentAreaNode &&
						'' + this.currentAreaNode.getAdcode() === '' + adcode)
				) {
					return;
				}

				const mapData = {
					type: 'FeatureCollection',
					features: geoJSON,
				};

				const ds = new DataSet();
				const geoDataView = ds.createView().source(mapData, {
					type: 'GeoJSON',
				}); // geoJSON 经纬度数据

				// 用户数据
				const dvData = ds.createView().source(this.list);
				dvData.transform({
					type: 'geo.region',
					field: 'name',
					geoDataView: geoDataView,
					as: ['longitude', 'lantitude'],
				});

				this.$data.geoData = geoDataView;
				this.$data.data = dvData;
				this.$data.name = name;

				if (callback) {
					callback(null, areaNode);
				}
			});
		},
	},
	mounted() {
		const self = this;
		// 高德API，需要在高德开放平台申请配置key
		$.getScript(
			'https://webapi.amap.com/maps?v=1.4.1&key=0d78256ea89beeb8c25d1cd047549d1f'
		)
			.then(() =>
				$.getScript('https://webapi.amap.com/ui/1.0/main.js?v=1.0.11')
			)
			.then(() => {
				// 调用高德 api 绘制底图以及获取 geo 数据
				const map = new AMap.Map('map');

				AMapUI.load(
					['ui/geo/DistrictExplorer', 'lib/$'],
					 (DistrictExplorer) => {
						// 创建一个实例
						const districtExplorer = (window.districtExplorer = new DistrictExplorer(
							{
								map: map,
							}
						));
						this.$data.districtExplorer = districtExplorer;
						// 根据省份编码渲染区域
						this.loadAreaNode(520000, function (error, areaNode) {
							if (error) {
								if (callback) {
									callback(error);
								}
								return;
							}
							window.currentAreaNode = areaNode;
							this.$data.currentAreaNode = areaNode;
							const node = _.cloneDeep(areaNode);
							districtExplorer.clearFeaturePolygons();
							districtExplorer.renderParentFeature(
								node,
								function (feature, i) {
									const fillColor = colors[i % colors.length];
									const strokeColor =
										colors[
											colors.length -
												1 -
												(i % colors.length)
										];

									return {
										cursor: 'default',
										bubble: true,
										strokeColor: strokeColor, //线颜色
										strokeOpacity: 1, //线透明度
										strokeWeight: 1, //线宽
										fillColor: fillColor, //填充色
										fillOpacity: 0.35, //填充透明度
									};
								}
							);
						});
					}
				);
			});
	},
};
</script>

<style lang="less" scoped>
.map-chart-container {
	width: 100%;
	.map {
		width: 100%;
		height: 665px;
	}
}
</style>
<style lang="less">
.g2-tooltip {
	position: absolute;
	min-height: 120px;
	background-color: rgba(0, 0, 0, 0.7);
	padding: 6px;
	z-index: 8;
	border-radius: 4px;
	.g2-tooltip-list {
		border-radius: 4px;
		background-color: rgb(0, 0, 0);
		padding: 12px 24px;
		text-align: center;
		color: #fff;
		font-size: 12px;
		b {
			display: block;
			line-height: 24px;
		}
		p {
			line-height: 24px;
		}
	}
}
</style>