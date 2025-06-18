// 引入 ECharts 库
import * as echarts from '../../components/ec-canvas/echarts';

Page({
    data: {
        phoneNumber: "",
        date: "",
        summary: {},
        lastMonthSummary: {},
        lastYearSummary: {},
        energyRatioPercent: 0,
        lastMonthRatioPercent: 0,
        lastYearRatioPercent: 0,
        compareMonth: {
            totalAmount: "--",
            energyCost: "--",
            energyRatio: "--"
        },
        compareYear: {
            totalAmount: "--",
            energyCost: "--",
            energyRatio: "--"
        },
        loadingInitial: false,

        barChart: {
            lazyLoad: true
        },
        pieChart: {
            lazyLoad: true
        },
        pieChartLastMonth: {
            lazyLoad: true
        },
        pieChartLastYear: {
            lazyLoad: true
        },
        energyRatioTrendChart: {
            lazyLoad: true
        },
        waterConsumptionTrendChart: {
            lazyLoad: true
        },
        electricityConsumptionTrendChart: {
            lazyLoad: true
        },
        gasConsumptionTrendChart: { // New chart for natural gas consumption
            lazyLoad: true
        },


        waterConsumptionTrendCharts: {}, // 存储每个酒店的水用量趋势图表实例
        electricityConsumptionTrendCharts: {}, // 存储每个酒店的电用量趋势图表实例 <--- 在这里添加
        energyRatioTrendData: [], // 存储能耗比趋势数据
        gasConsumptionTrendCharts: {}, // 每家酒店的天然气用量图表
        energyCostPieCharts: {}, // 每家酒店能耗费用占比饼图

        
        energyRatioTrendCategories: [], // 存储能耗比趋势月份
        waterConsumptionTrendData: [], // 存储水用量趋势数据
        waterConsumptionTrendCategories: [], // 存储水用量趋势月份
        electricityConsumptionTrendData: [], // 存储电用量趋势数据
        electricityConsumptionTrendCategories: [], // 存储电用量趋势月份
        gasConsumptionTrendData: [], // New: 存储气用量趋势数据
        gasConsumptionTrendCategories: [], // New: 存储气用量趋势月份

        tabList: ['本月', '上月', '去年'],
        activeTab: 0,

        top5List: [],
        compareTable: [],
        hotelList: [], // Added for the new hotel list display
        totalSummary: { // Added for overall totals
            totalAmount: "--",
            energyCost: "--",
            energyRatio: "--"
        },
    },

    switchTab(e) {
        const index = Number(e.currentTarget.dataset.index);
        this.setData({
            activeTab: index
        }, () => {
            if (index === 0) {
                this.drawPieChart(this.data.summary, 'current');
            } else if (index === 1) {
                this.drawPieChart(this.data.lastMonthSummary, 'lastMonth');
            } else if (index === 2) {
                this.drawPieChart(this.data.lastYearSummary, 'lastYear');
            }
        });
    },


    switchHotelTrendTab(e) {
      const hotelCode = e.currentTarget.dataset.hotelcode;
      const tab = e.currentTarget.dataset.tab;
    
      const hotelList = this.data.hotelList.map(hotel => {
        if (hotel.hotelCode === hotelCode) {
          hotel.activeTrendTab = tab;
        }
        return hotel;
      });
    
      this.setData({ hotelList }, () => {
        if (tab === 'water') {
          this.loadWaterDataAndInitChart(hotelCode);
        } else if (tab === 'electricity') {
          this.loadElectricityDataAndInitChart(hotelCode);
        } else if (tab === 'gas') {
          this.loadGasDataAndInitChart(hotelCode);
        }
      });
    },
    
    
    

    onLoad() {
        const phone = wx.getStorageSync("phoneNumber");
        if (!phone) {
            wx.showToast({
                title: "请先登录",
                icon: "none"
            });
            wx.reLaunch({
                url: "/pages/index/index"
            });
            return;
        }
        const date = this.getPreviousDate();
        this.setData({
            phoneNumber: phone,
            date
        }, this.queryData);
        wx.showShareMenu({
            withShareTicket: true,
            menus: ['shareAppMessage', 'shareTimeline']
        });
    },

    getPreviousDate() {
        const now = new Date();
        now.setMonth(now.getMonth() - 1);
        const year = now.getFullYear();
        const month = String(now.getMonth() + 1).padStart(2, '0');
        return `${year}-${month}`;
    },

    bindDateChange(e) {
        this.setData({
            date: e.detail.value
        }, this.queryData);
    },

    goBack() {
        wx.navigateBack();
    },

    queryData() {
        this.setData({
            loadingInitial: true
        });
        let electricityConsumptionTrendCharts = {};
        let gasConsumptionTrendCharts = {};
        let energyCostPieCharts = {};

        const [year, month] = this.data.date.split("-");
        const startDate = `${year}-${month}-01`;
        const daysInMonth = new Date(year, month, 0).getDate();
        const endDate = `${year}-${month}-${daysInMonth}`;
        const currentYear = parseInt(year);
        const currentMonth = parseInt(month);
        const trendPromises = [];
        const energyRatioTrendCategories = [];
        const waterConsumptionTrendCategories = [];
        const electricityConsumptionTrendCategories = [];
        const gasConsumptionTrendCategories = []; // New: Categories for gas trend

        for (let i = 1; i <= currentMonth; i++) {
            const trendMonth = String(i).padStart(2, '0');
            const trendStartDate = `${currentYear}-${trendMonth}-01`;
            const trendDaysInMonth = new Date(currentYear, i, 0).getDate();
            const trendEndDate = `${currentYear}-${trendMonth}-${trendDaysInMonth}`;
            energyRatioTrendCategories.push(`${currentYear}-${trendMonth}`);
            waterConsumptionTrendCategories.push(`${currentYear}-${trendMonth}`);
            electricityConsumptionTrendCategories.push(`${currentYear}-${trendMonth}`);
            gasConsumptionTrendCategories.push(`${currentYear}-${trendMonth}`);
            // New: Add to gas trend categories

            trendPromises.push(
                new Promise((resolve, reject) => {
                    wx.request({
                        url: 'https://api.cbkj.net.cn/getTotalEnergyConsumption_Monthly_Report.php',
                        method: 'GET',
                        data: {
                            startDate: trendStartDate,
                            endDate: trendEndDate
                        },
                        success: (res) => {
                            // Extract values directly, assume energyRatio is already a ratio
                            const energyRatio = res.data?.data?.current?.summary?.energyRatio;
                            const waterPrice = res.data?.data?.current?.summary?.WATER_PRICE;
                            const electricityPrice = res.data?.data?.current?.summary?.ELECTRICITY_PRICE;
                            const naturalGasPrice = res.data?.data?.current?.summary?.NATURALGAS_PRICE; // New: Get natural gas price
                            resolve({
                                energyRatio: typeof energyRatio === 'number' ? energyRatio.toFixed(2) : "0.00",
                                waterPrice: typeof waterPrice === 'number' ? waterPrice.toFixed(2) : "0.00",
                                electricityPrice: typeof electricityPrice === 'number' ? electricityPrice.toFixed(2) : "0.00",
                                naturalGasPrice: typeof naturalGasPrice === 'number' ? naturalGasPrice.toFixed(2) : "0.00"
                            });
                        },
                        fail: (err) => {
                            console.error(`Failed to fetch data for ${trendStartDate}:`, err);
                            resolve({
                                energyRatio: "0.00",
                                waterPrice: "0.00",
                                electricityPrice: "0.00",
                                naturalGasPrice: "0.00"
                            });
                        } // If fails, treat as 0
                    });
                })
            );
        }

        Promise.all([
                new Promise((resolve, reject) => {
                    wx.request({
                        url: 'https://api.cbkj.net.cn/getTotalEnergyConsumption_Monthly_Report.php',
                        method: 'GET',
                        data: {
                            startDate,
                            endDate
                        },
                        success: resolve,
                        fail: reject
                    });
                }),
                ...trendPromises
            ])
            .then((results) => {
                const mainRes = results[0];
                const monthlyTrendData = results.slice(1); // Array of { energyRatio, waterPrice, electricityPrice, naturalGasPrice }

                const energyRatioTrendData = monthlyTrendData.map(item => item.energyRatio);
                const waterConsumptionTrendData = monthlyTrendData.map(item => item.waterPrice);
                const electricityConsumptionTrendData = monthlyTrendData.map(item => item.electricityPrice);
                const gasConsumptionTrendData = monthlyTrendData.map(item => item.naturalGasPrice); // New: Extract gas price data

                const current = mainRes.data?.data?.current?.summary || {};
                const lastMonth = mainRes.data?.data?.lastMonth?.summary || {};
                const lastYear = mainRes.data?.data?.lastYear?.summary || {};

                // *********** 获取上月和去年的酒店列表数据 ***********
                const lastMonthHotelListRaw = mainRes.data?.data?.lastMonth?.list || [];
                const lastYearHotelListRaw = mainRes.data?.data?.lastYear?.list || [];

                // 辅助函数：根据 hotelCode 查找对应酒店的数据
                const findHotelData = (hotelList, hotelCode) => {
                    return hotelList.find(hotel => hotel.hotelCode === hotelCode);
                };
                // ******************************************************

                const ratio = (val, prev) => {
                    if (prev === 0 || prev === undefined || prev === null) return "--";
                    if (val === undefined || val === null) return "--";
                    return (((val - prev) / prev) * 100).toFixed(1);
                };

                // *********** 修改后的 parseRatio 函数 (包含 +/- 符号) ***********
                const parseRatio = (val) => {
                    if (val === "--") return {
                        value: val,
                        abs: 0
                    };
                    const num = parseFloat(val);
                    const sign = num >= 0 ? '+' : ''; // 如果是正数或0，加一个 '+' 符号
                    return {
                        value: `${sign}${val}%`, // 在百分比数值前加上符号
                        abs: Math.abs(num)
                    };
                };
                // *************************************************

                const rMonthTotal = parseRatio(ratio(current.totalAmount, lastMonth.totalAmount));
                const rYearTotal = parseRatio(ratio(current.totalAmount, lastYear.totalAmount));

                const hotelListRaw = mainRes.data?.data?.current?.list || [];
                let totalAmountSum = 0;
                let energyCostSum = 0;

                const hotelListFormatted = hotelListRaw.map(item => {
                  const totalAmount = parseFloat(item.totalAmount || 0);
                  const energyCost = parseFloat(item.energyCost || 0);
                  const energyRatio = parseFloat(item.energyRatio || 0);
                  const occ = parseFloat(item.occ || 0);
                  // const occ = parseFloat(item.occ || 0) * 100;

                  const soldRooms = parseInt(item.soldRooms || 0);
              
                  totalAmountSum += totalAmount;
                  energyCostSum += energyCost;
              
                  const lastMonthHotel = findHotelData(lastMonthHotelListRaw, item.hotelCode) || {};
                  const lastYearHotel = findHotelData(lastYearHotelListRaw, item.hotelCode) || {};
              
                  const parseRatio = (val) => {
                      if (val === "--") return { value: val, abs: 0 };
                      const num = parseFloat(val);
                      const sign = num >= 0 ? "+" : "";
                      return { value: `${sign}${val}%`, abs: Math.abs(num) };
                  };
              
                  const ratio = (val, prev) => {
                      if (prev === 0 || prev === undefined || prev === null) return "--";
                      if (val === undefined || val === null) return "--";
                      return (((val - prev) / prev) * 100).toFixed(1);
                  };
              
                  // 比较值（环比/同比）
                  const rMonthTotal = parseRatio(ratio(totalAmount, lastMonthHotel.totalAmount));
                  const rYearTotal = parseRatio(ratio(totalAmount, lastYearHotel.totalAmount));
              
                  const rMonthEnergyCost = parseRatio(ratio(energyCost, lastMonthHotel.energyCost));
                  const rYearEnergyCost = parseRatio(ratio(energyCost, lastYearHotel.energyCost));
              
                  const rMonthEnergyRatio = parseRatio(ratio(energyRatio, lastMonthHotel.energyRatio));
                  const rYearEnergyRatio = parseRatio(ratio(energyRatio, lastYearHotel.energyRatio));
              
                  const rMonthOcc = parseRatio(ratio(occ, lastMonthHotel.occ));
                  const rYearOcc = parseRatio(ratio(occ, lastYearHotel.occ));
              
                  const rMonthSold = parseRatio(ratio(soldRooms, lastMonthHotel.soldRooms));
                  const rYearSold = parseRatio(ratio(soldRooms, lastYearHotel.soldRooms));
              
                  return {
                      hotelCode: item.hotelCode,
                      DepName: item.DepName || "N/A",
              
                      // 旧功能保留
                      totalAmount: totalAmount.toFixed(2),
                      energyCost: energyCost.toFixed(2),
                      energyRatio: energyRatio.toFixed(2) + "%",
              
                      // 新增完整结构
                      current: {
                          totalAmount: totalAmount.toFixed(2),
                          energyCost: energyCost.toFixed(2),
                          energyRatio: energyRatio.toFixed(2) + "%",
                          occ: occ.toFixed(2) + "%",
                          soldRooms
                      },
                      lastMonth: {
                          totalAmount: lastMonthHotel.totalAmount !== undefined ? parseFloat(lastMonthHotel.totalAmount).toFixed(2) : "--",
                          energyCost: lastMonthHotel.energyCost !== undefined ? parseFloat(lastMonthHotel.energyCost).toFixed(2) : "--",
                          energyRatio: lastMonthHotel.energyRatio !== undefined ? parseFloat(lastMonthHotel.energyRatio).toFixed(2) + "%" : "--",
                          occ: lastMonthHotel.occ !== undefined ? parseFloat(lastMonthHotel.occ).toFixed(2) + "%" : "--",
                          soldRooms: lastMonthHotel.soldRooms ?? "--"
                      },
                      lastYear: {
                          totalAmount: lastYearHotel.totalAmount !== undefined ? parseFloat(lastYearHotel.totalAmount).toFixed(2) : "--",
                          energyCost: lastYearHotel.energyCost !== undefined ? parseFloat(lastYearHotel.energyCost).toFixed(2) : "--",
                          energyRatio: lastYearHotel.energyRatio !== undefined ? parseFloat(lastYearHotel.energyRatio).toFixed(2) + "%" : "--",
                          occ: lastYearHotel.occ !== undefined ? parseFloat(lastYearHotel.occ).toFixed(2) + "%" : "--",
                          soldRooms: lastYearHotel.soldRooms ?? "--"
                      },
                      compareMonth: {
                          totalAmount: rMonthTotal,
                          energyCost: rMonthEnergyCost,
                          energyRatio: rMonthEnergyRatio,
                          occ: rMonthOcc,
                          soldRooms: rMonthSold
                      },
                      compareYear: {
                          totalAmount: rYearTotal,
                          energyCost: rYearEnergyCost,
                          energyRatio: rYearEnergyRatio,
                          occ: rYearOcc,
                          soldRooms: rYearSold
                      }
                  };
              });
              

                // Calculate the overall energy ratio for the total summary
                let totalEnergyRatioValue = "0.00";
                if (totalAmountSum > 0) {
                    totalEnergyRatioValue = ((energyCostSum / totalAmountSum) * 100).toFixed(2);
                }

                const totalSummary = {
                    totalAmount: totalAmountSum.toFixed(2),
                    energyCost: energyCostSum.toFixed(2),
                    energyRatio: totalEnergyRatioValue + '%'
                };

// 在这里插入处理 hotelTrendsData 的代码块 START
let hotelList = hotelListFormatted; // 使用已经格式化好的 hotelListFormatted
let waterConsumptionTrendCharts = {};

// 从 mainRes.data.data 中获取 hotelTrends
const hotelTrendsData = mainRes.data.data.hotelTrends || [];

hotelList.forEach(hotel => {
    const trend = hotelTrendsData.find(item => item.hotelCode === hotel.hotelCode);
    if (trend && trend.monthlyData) {
        hotel.waterConsumptionTrendCategories = trend.monthlyData.map(item => item.month.substring(5));
        hotel.waterConsumptionTrendData = trend.monthlyData.map(item => item.WATER_USAGE);
        waterConsumptionTrendCharts[hotel.hotelCode] = { lazyLoad: true };
        hotel.electricityConsumptionTrendCategories = trend.monthlyData.map(item => item.month.substring(5));
        hotel.electricityConsumptionTrendData = trend.monthlyData.map(item => item.ELECTRICITY_USAGE);
        electricityConsumptionTrendCharts[hotel.hotelCode] = { lazyLoad: true }; // ✅ 加在这里
        hotel.gasConsumptionTrendCategories = trend.monthlyData.map(item => item.month.substring(5));
        hotel.gasConsumptionTrendData = trend.monthlyData.map(item => item.NATURALGAS_USAGE);
        gasConsumptionTrendCharts[hotel.hotelCode] = { lazyLoad: true };
        hotel.activeTrendTab = 'water';
        // 饼图数据
        if (trend && trend.monthlyData && trend.monthlyData.length > 0) {
          const latestMonth = trend.monthlyData[trend.monthlyData.length - 1]; // 最新月份数据

          hotel.energyCostPieData = [
            { value: latestMonth.WATER_PRICE || 0, name: '水费' },
            { value: latestMonth.ELECTRICITY_PRICE || 0, name: '电费' },
            { value: latestMonth.NATURALGAS_PRICE || 0, name: '燃气费' }
          ];

          energyCostPieCharts[hotel.hotelCode] = { lazyLoad: true };
        }
    } else {
        hotel.waterConsumptionTrendCategories = [];
        hotel.waterConsumptionTrendData = [];
    }
    // 为每个酒店的图表设置lazyLoad属性
    waterConsumptionTrendCharts[hotel.hotelCode] = { lazyLoad: true };
});
// 在这里插入处理 hotelTrendsData 的代码块 END



                const top5 = hotelListRaw
                    .filter(item => typeof item.energyRatio === 'number')
                    .sort((a, b) => b.energyRatio - a.energyRatio)
                    .slice(0, 5)
                    .map((item, index) => ({
                        ...item,
                        rank: index + 1,
                        energyRatioFixed: item.energyRatio.toFixed(2) // Keeping toFixed(2) for display
                    }));

                const createCompareTable = (current, lastMonth, lastYear) => {
                    const fields = ['totalAmount', 'occ', 'soldRooms', 'energyCost', 'energyRatio'];
                    const labels = {
                        totalAmount: '总营收(万元)',
                        occ: '出租率',
                        soldRooms: '已卖房(间)',
                        energyCost: '能耗费(万元)',
                        energyRatio: '能耗比'
                    };
                    const formatValue = (val, isPercent = false) => {
                        if (val === null || val === undefined) return "--";
                        if (typeof val === "number") {
                            if (isPercent) return val.toFixed(2) + '%';
                            // 能耗比和出租率直接显示百分比
                            return val.toFixed(2);
                        }
                        return val;
                    };

                    const calcRatio = (cur, prev) => {
                        if (prev === 0 || prev === undefined || prev === null) return "--";
                        if (cur === undefined || cur === null) return "--";
                        return (((cur - prev) / prev) * 100).toFixed(1) + '%';
                    };

                    return fields.map(field => ({
                        label: labels[field],
                        current: formatValue(current[field], field === 'energyRatio' || field === 'occ'),
                        lastMonth: formatValue(lastMonth[field], field === 'energyRatio' || field === 'occ'),
                        lastYear: formatValue(lastYear[field], field === 'energyRatio' || field === 'occ'),
                        monthCompare: calcRatio(current[field], lastMonth[field]),
                        yearCompare: calcRatio(current[field], lastYear[field])
                    }));
                };

                this.setData({
                    summary: current,
                    lastMonthSummary: lastMonth,
                    lastYearSummary: lastYear,
                    // 能耗比直接使用原始值，只格式化为字符串
                    energyRatioPercent: current.energyRatio ? current.energyRatio.toFixed(2) : "0.00",
                    lastMonthRatioPercent: lastMonth.energyRatio ? lastMonth.energyRatio.toFixed(2) : "0.00",
                    lastYearRatioPercent: lastYear.energyRatio ? lastYear.energyRatio.toFixed(2) : "0.00",
                    top5List: top5,
                    compareTable: createCompareTable(current, lastMonth, lastYear),
                    energyRatioTrendData: energyRatioTrendData, // 趋势数据不再乘以100
                    energyRatioTrendCategories: energyRatioTrendCategories,
                    waterConsumptionTrendData: waterConsumptionTrendData,
                    waterConsumptionTrendCategories: waterConsumptionTrendCategories,
                    electricityConsumptionTrendData: electricityConsumptionTrendData,
                    electricityConsumptionTrendCategories: electricityConsumptionTrendCategories,
                    gasConsumptionTrendData: gasConsumptionTrendData, // New: Set gas trend data
                    gasConsumptionTrendCategories: gasConsumptionTrendCategories, // New: Set gas trend categories
                    // hotelList: hotelListFormatted, // Set the formatted hotel list
                    hotelList: hotelList, // 使用上面处理过的 hotelList 变量
                    waterConsumptionTrendCharts: waterConsumptionTrendCharts, // 设置水用量趋势图表
                    electricityConsumptionTrendCharts: electricityConsumptionTrendCharts, // ✅ 必须
                    gasConsumptionTrendCharts: gasConsumptionTrendCharts,
                    energyCostPieCharts: energyCostPieCharts,
                    totalSummary: totalSummary, // Set the calculated total summary

                    compareMonth: {
                        totalAmount: rMonthTotal,
                        energyCost: parseRatio(ratio(current.energyCost, lastMonth.energyCost)),
                        energyRatio: parseRatio(ratio(current.energyRatio, lastMonth.energyRatio))
                    },
                    compareYear: {
                        totalAmount: rYearTotal,
                        energyCost: parseRatio(ratio(current.energyCost, lastYear.energyCost)),
                        energyRatio: parseRatio(ratio(current.energyRatio, lastYear.energyRatio))
                    }
                }, () => {
                    this.drawBarChart(current, lastMonth, lastYear);
                    this.drawPieChart(current, 'current');
                    this.drawPieChart(lastMonth, 'lastMonth');
                    this.drawPieChart(lastYear, 'lastYear');
                    this.drawEnergyRatioTrendChart();
                    this.drawWaterConsumptionTrendChart();
                    this.drawElectricityConsumptionTrendChart();
                    this.drawGasConsumptionTrendChart(); // New: Draw gas consumption chart

                    // 在这里循环初始化每个酒店的水用量趋势图表 START
                    hotelList.forEach(hotel => {
                      // 只有当有趋势数据时才尝试初始化图表
                      if (hotel.waterConsumptionTrendCategories && hotel.waterConsumptionTrendCategories.length > 0) {
                          this.selectComponent(`#waterConsumptionTrendChart-${hotel.hotelCode}`).init((canvas, width, height, dpr) => {
                              return this.initWaterConsumptionTrendChart(canvas, width, height, dpr, hotel.waterConsumptionTrendCategories, hotel.waterConsumptionTrendData);
                          });
                      }
                      
                      if (hotel.electricityConsumptionTrendCategories && hotel.electricityConsumptionTrendCategories.length > 0) {
                        this.selectComponent(`#electricityConsumptionTrendChart-${hotel.hotelCode}`).init((canvas, width, height, dpr) => {
                            return this.initElectricityConsumptionTrendChart(canvas, width, height, dpr, hotel.electricityConsumptionTrendCategories, hotel.electricityConsumptionTrendData);
                        });
                    }

                    if (hotel.gasConsumptionTrendCategories && hotel.gasConsumptionTrendCategories.length > 0) {
                      this.selectComponent(`#gasConsumptionTrendChart-${hotel.hotelCode}`)?.init((canvas, width, height, dpr) => {
                        return this.initGasConsumptionTrendChart(
                          canvas, width, height, dpr,
                          hotel.gasConsumptionTrendCategories,
                          hotel.gasConsumptionTrendData
                        );
                      });
                    }
                    
                    if (hotel.energyCostPieData && hotel.energyCostPieData.length > 0) {
                      this.selectComponent(`#energyCostPieChart-${hotel.hotelCode}`)?.init((canvas, width, height, dpr) => {
                        return this.initEnergyCostPieChart(canvas, width, height, dpr, hotel.energyCostPieData);
                      });
                    }
                    

                  });
                  
                });
            })
            .catch((err) => {
                console.error("Query data failed:", err);
                wx.showToast({
                    title: "加载失败",
                    icon: "none"
                });
            })
            .finally(() => {
                this.setData({
                    loadingInitial: false
                });
            });
    },

    drawBarChart(current, lastMonth, lastYear) {
        const chartComponent = this.selectComponent('#barChart');
        if (!chartComponent) return;

        chartComponent.init((canvas, width, height, dpr) => {
            const chart = echarts.init(canvas, null, {
                width,
                height,
                devicePixelRatio: dpr
            });
            canvas.setChart(chart);

            chart.setOption({
                title: {
                    text: '营收能耗费用同比环比',
                    left: 'center',
                    top: 10,
                    textStyle: {
                        fontSize: 14
                    }
                },
                tooltip: {
                    trigger: 'axis',
                    confine: true,
                    backgroundColor: '#fff',
                    textStyle: {
                        color: '#000'
                    },
                    formatter(params) {
                        let result = `${params[0].axisValue}\n`;
                        params.forEach(item => {
                            let val = item.value;
                            // 能耗比直接加上百分号，不进行乘以100操作
                            val = item.seriesName.includes('能耗比') ? parseFloat(val).toFixed(2) + '%' : val.toFixed(2) + ' 万元';
                            result += `${item.marker}${item.seriesName}: ${val}\n`;
                        });
                        return result;
                    }
                },
                legend: {
                    data: ['总营业额', '能耗费用', '能耗比'],
                    top: 35
                },
                grid: {
                    top: 100,
                    bottom: 20,
                    left: 10,
                    right: 10,
                    containLabel: true
                },
                xAxis: {
                    type: 'category',
                    data: ['本期', '上月', '去年']
                },
                yAxis: [{
                    type: 'value',
                    name: '金额 (万元)',
                    position: 'left'
                }, {
                    type: 'value',
                    name: '能耗比 (%)',
                    position: 'right',
                    axisLabel: {
                        formatter: '{value}%'
                    }
                }],
                series: [{
                    name: '总营业额',
                    type: 'bar',
                    barWidth: 20,
                    data: [current.totalAmount || 0, lastMonth.totalAmount || 0, lastYear.totalAmount || 0],
                    yAxisIndex: 0
                }, {
                    name: '能耗费用',
                    type: 'bar',
                    barWidth: 20,
                    data: [current.energyCost || 0, lastMonth.energyCost || 0, lastYear.energyCost || 0],
                    yAxisIndex: 0
                }, {
                    name: '能耗比',
                    type: 'line',
                    smooth: true,
                    // 能耗比数据直接传递，不乘以100
                    data: [
                        current.energyRatio || 0,
                        lastMonth.energyRatio || 0,
                        lastYear.energyRatio || 0
                    ],
                    yAxisIndex: 1
                }]
            });
            return chart;
        });
    },

    drawPieChart(data, type = 'current') {
        let id = '#pieChart';
        let title = '本月能耗费用构成';

        if (type === 'lastMonth') {
            id = '#pieChartLastMonth';
            title = '上月能耗费用构成';
        } else if (type === 'lastYear') {
            id = '#pieChartLastYear';
            title = '去年同期能耗费用构成';
        }

        const chartComponent = this.selectComponent(id);
        if (!chartComponent) return;
        chartComponent.init((canvas, width, height, dpr) => {
            const chart = echarts.init(canvas, null, {
                width,
                height,
                devicePixelRatio: dpr
            });
            canvas.setChart(chart);

            const water = data.WATER_PRICE || 0;
            const electricity = data.ELECTRICITY_PRICE || 0;
            const gas = data.NATURALGAS_PRICE || 0;
            const totalCost = data.energyCost || 0;

            chart.setOption({
                title: [{
                    text: title,
                    left: 'center',
                    top: 10,
                    textStyle: {
                        fontSize: 14
                    }
                }, {
                    text: '总费用',
                    subtext: `${totalCost.toFixed(2)} 万元`,
                    left: 'center',
                    top: '45%',
                    textStyle: {
                        fontSize: 14,
                        color: '#000'
                    },
                    subtextStyle: {
                        fontSize: 14,
                        color: '#000'
                    }
                }],
                tooltip: {
                    trigger: 'item',
                    confine: true,
                    formatter: '{b}: {c} 万元 ({d}%)'
                },
                legend: {
                    orient: 'horizontal',
                    bottom: 1,
                    data: ['水费', '电费', '燃气费']
                },
                series: [{
                    name: '能耗费用',
                    type: 'pie',
                    radius: ['40%', '60%'],
                    center: ['50%', '55%'],
                    data: [{
                        value: water,
                        name: '水费'
                    }, {
                        value: electricity,
                        name: '电费'
                    }, {
                        value: gas,
                        name: '燃气费'
                    }],
                    label: {
                        formatter: '{b}\n{d}%',
                        fontSize: 12
                    }
                }]
            });
            return chart;
        });
    },

    drawEnergyRatioTrendChart() {
        const chartComponent = this.selectComponent('#energyRatioTrendChart');
        if (!chartComponent) return;

        chartComponent.init((canvas, width, height, dpr) => {
            const chart = echarts.init(canvas, null, {
                width,
                height,
                devicePixelRatio: dpr
            });
            canvas.setChart(chart);

            const {
                energyRatioTrendCategories,
                energyRatioTrendData
            } = this.data;

            chart.setOption({
                title: {
                    text: '本年能耗比趋势',
                    left: 'center',
                    top: 10,
                    textStyle: {
                        fontSize: 14
                    }
                },
                tooltip: {
                    trigger: 'axis',
                    confine: true,
                    backgroundColor: '#fff',
                    textStyle: {
                        color: '#000'
                    },
                    formatter(params) {
                        let result = `${params[0].axisValue}\n`;
                        params.forEach(item => {
                            // 能耗比直接加上百分号，不进行乘以100操作
                            result += `${item.marker}${item.seriesName}: ${parseFloat(item.value).toFixed(2)}% \n`;
                        });
                        return result;
                    }
                },
                grid: {
                    top: 70,
                    bottom: 20,
                    left: 10,
                    right: 10,
                    containLabel: true
                },
                xAxis: {
                    type: 'category',
                    data: energyRatioTrendCategories.map(date => date.substring(5)),
                    axisLabel: {
                        interval: 0,
                        rotate: 45
                    }
                },
                yAxis: {
                    type: 'value',
                    name: '能耗比 (%)',
                    axisLabel: {
                        formatter: '{value}%' // Y轴标签直接加百分号
                    }
                },
                series: [{
                    name: '能耗比',
                    type: 'line',
                    smooth: true,
                    data: energyRatioTrendData, // 数据直接使用原始值
                    itemStyle: {
                        color: '#5470C6'
                    }
                }]
            });
            return chart;
        });
    },

    drawWaterConsumptionTrendChart() {
        const chartComponent = this.selectComponent('#waterConsumptionTrendChart');
        if (!chartComponent) return;

        chartComponent.init((canvas, width, height, dpr) => {
            const chart = echarts.init(canvas, null, {
                width,
                height,
                devicePixelRatio: dpr
            });
            canvas.setChart(chart);

            const {
                waterConsumptionTrendCategories,
                waterConsumptionTrendData
            } = this.data;

            chart.setOption({
                title: {
                    text: '本年水费用趋势',
                    left: 'center',
                    top: 10,
                    textStyle: {
                        fontSize: 14
                    }
                },
                tooltip: {
                    trigger: 'axis',
                    confine: true,
                    backgroundColor: '#fff',
                    textStyle: {
                        color: '#000'
                    },
                    formatter(params) {
                        let result = `${params[0].axisValue}\n`;
                        params.forEach(item => {
                            result += `${item.marker}${item.seriesName}: ${parseFloat(item.value).toFixed(2)} 万元\n`;
                        });
                        return result;
                    }
                },
                grid: {
                    top: 70,
                    bottom: 20,
                    left: 10,
                    right: 10,
                    containLabel: true
                },
                xAxis: {
                    type: 'category',
                    data: waterConsumptionTrendCategories.map(date => date.substring(5)),
                    axisLabel: {
                        interval: 0,
                        rotate: 45
                    }
                },
                yAxis: {
                    type: 'value',
                    name: '水费用 (万元))',
                    axisLabel: {
                        formatter: '{value}万元'
                    }
                },
                series: [{
                    name: '水费用',
                    type: 'line',
                    smooth: true,
                    data: waterConsumptionTrendData,
                    itemStyle: {
                        color: '#7CBB40' // You can choose a different color for this line
                    }
                }]
            });
            return chart;
        });
    },

    drawElectricityConsumptionTrendChart() {
        const chartComponent = this.selectComponent('#electricityConsumptionTrendChart');
        if (!chartComponent) return;

        chartComponent.init((canvas, width, height, dpr) => {
            const chart = echarts.init(canvas, null, {
                width,
                height,
                devicePixelRatio: dpr
            });
            canvas.setChart(chart);

            const {
                electricityConsumptionTrendCategories,
                electricityConsumptionTrendData
            } = this.data;

            chart.setOption({
                title: {
                    text: '本年电费用趋势',
                    left: 'center',
                    top: 10,
                    textStyle: {
                        fontSize: 14
                    }
                },
                tooltip: {
                    trigger: 'axis',
                    confine: true,
                    backgroundColor: '#fff',
                    textStyle: {
                        color: '#000'
                    },
                    formatter(params) {
                        let result = `${params[0].axisValue}\n`;
                        params.forEach(item => {
                            result += `${item.marker}${item.seriesName}: ${parseFloat(item.value).toFixed(2)} 万元\n`;
                        });
                        return result;
                    }
                },
                grid: {
                    top: 70,
                    bottom: 20,
                    left: 10,
                    right: 10,
                    containLabel: true
                },
                xAxis: {
                    type: 'category',
                    data: electricityConsumptionTrendCategories.map(date => date.substring(5)),
                    axisLabel: {
                        interval: 0,
                        rotate: 45
                    }
                },
                yAxis: {
                    type: 'value',
                    name: '电费用 (万元))',
                    axisLabel: {
                        formatter: '{value}万元'
                    }
                },
                series: [{
                    name: '电费用',
                    type: 'line',
                    smooth: true,
                    data: electricityConsumptionTrendData,
                    itemStyle: {
                        color: '#F4A261' // You can choose a different color for this line
                    }
                }]
            });
            return chart;
        });
    },

    // New: drawGasConsumptionTrendChart function
    drawGasConsumptionTrendChart() {
        const chartComponent = this.selectComponent('#gasConsumptionTrendChart');
        if (!chartComponent) return;

        chartComponent.init((canvas, width, height, dpr) => {
            const chart = echarts.init(canvas, null, {
                width,
                height,
                devicePixelRatio: dpr
            });
            canvas.setChart(chart);

            const {
                gasConsumptionTrendCategories,
                gasConsumptionTrendData
            } = this.data;

            chart.setOption({
                title: {
                    text: '本年天然气费用趋势',
                    left: 'center',
                    top: 10,
                    textStyle: {
                        fontSize: 14
                    }
                },
                tooltip: {
                    trigger: 'axis',
                    confine: true,
                    backgroundColor: '#fff',
                    textStyle: {
                        color: '#000'
                    },
                    formatter(params) {
                        let result = `${params[0].axisValue}\n`;
                        params.forEach(item => {
                            result += `${item.marker}${item.seriesName}: ${parseFloat(item.value).toFixed(2)} 万元\n`;
                        });
                        return result;
                    }
                },
                grid: {
                    top: 70,
                    bottom: 20,
                    left: 10,
                    right: 10,
                    containLabel: true
                },
                xAxis: {
                    type: 'category',
                    data: gasConsumptionTrendCategories.map(date => date.substring(5)),
                    axisLabel: {
                        interval: 0,
                        rotate: 45
                    }
                },
                yAxis: {
                    type: 'value',
                    name: '天然气费用 (万元)',
                    axisLabel: {
                        formatter: '{value}万元'
                    }
                },
                series: [{
                    name: '天然气费用',
                    type: 'line',
                    smooth: true,
                    data: gasConsumptionTrendData,
                    itemStyle: {
                        color: '#91CC75' // You can choose a different color for this line
                    }
                }]
            });
            return chart;
        });
    },

// 酒店水趋势
    initWaterConsumptionTrendChart: function (canvas, width, height, dpr, waterConsumptionTrendCategories, waterConsumptionTrendData) {
          const chart = echarts.init(canvas, null, {
          width: width,
          height: height,
          devicePixelRatio: dpr
      });
      canvas.setChart(chart);

      const option = {
          title: {
              text: '水用量趋势',
              left: 'center',
              textStyle: {
                  fontSize: 14
              }
          },
          tooltip: {
              trigger: 'axis',
              formatter: function (params) {
                  let result = `${params[0].axisValue}\n`;
                  params.forEach(item => {
                      result += `${item.marker}${item.seriesName}: ${parseFloat(item.value).toFixed(2)} 吨\n`;
                  });
                  return result;
              }
          },
          grid: {
              top: 70,
              bottom: 20,
              left: 10,
              right: 10,
              containLabel: true
          },
          xAxis: {
              type: 'category',
              data: waterConsumptionTrendCategories.map(date => date.substring(0)),
              axisLabel: {
                  interval: 0,
                  rotate: 45
              }
          },
          yAxis: {
              type: 'value',
              name: '水用量 (吨)',
              axisLabel: {
                  formatter: '{value}吨'
              }
          },
          series: [{
              name: '水用量',
              type: 'line',
              smooth: true,
              data: waterConsumptionTrendData,
              itemStyle: {
                  color: '#5470C6' // 可以选择不同的颜色
              }
          }]
      };
      chart.setOption(option);
      return chart;
  },


// 酒店电用量趋势
initElectricityConsumptionTrendChart: function (canvas, width, height, dpr, categories, data) {
  const chart = echarts.init(canvas, null, {
      width: width,
      height: height,
      devicePixelRatio: dpr
  });
  canvas.setChart(chart);

  const option = {
      title: {
          text: '电用量趋势',
          left: 'center',
          textStyle: {
              fontSize: 14
          }
      },
      tooltip: {
          trigger: 'axis',
          formatter: function (params) {
              let result = `${params[0].axisValue}\n`;
              params.forEach(item => {
                  result += `${item.marker}${item.seriesName}: ${parseFloat(item.value).toFixed(2)} 度\n`;
              });
              return result;
          }
      },
      grid: {
          top: 70,
          bottom: 20,
          left: 10,
          right: 10,
          containLabel: true
      },
      xAxis: {
          type: 'category',
          data: categories.map(date => date),
          axisLabel: {
              interval: 0,
              rotate: 45
          }
      },
      yAxis: {
          type: 'value',
          name: '电用量 (度)',
          axisLabel: {
              formatter: '{value}度'
          }
      },
      series: [{
          name: '电用量',
          type: 'line',
          smooth: true,
          data: data,
          itemStyle: {
              color: '#F4A261'
          }
      }]
  };
  chart.setOption(option);
  return chart;
},
// 酒店气用量趋势
initGasConsumptionTrendChart: function (canvas, width, height, dpr, categories, data) {
  const chart = echarts.init(canvas, null, {
      width: width,
      height: height,
      devicePixelRatio: dpr
  });
  canvas.setChart(chart);

  const option = {
      title: {
          text: '天然气用量趋势',
          left: 'center',
          textStyle: {
              fontSize: 14
          }
      },
      tooltip: {
          trigger: 'axis',
          formatter: function (params) {
              let result = `${params[0].axisValue}\n`;
              params.forEach(item => {
                  result += `${item.marker}${item.seriesName}: ${parseFloat(item.value).toFixed(2)} 立方米\n`;
              });
              return result;
          }
      },
      grid: {
          top: 70,
          bottom: 20,
          left: 10,
          right: 10,
          containLabel: true
      },
      xAxis: {
          type: 'category',
          data: categories.map(date => date),
          axisLabel: {
              interval: 0,
              rotate: 45
          }
      },
      yAxis: {
          type: 'value',
          name: '天然气 (立方米)',
          axisLabel: {
              formatter: '{value}m³'
          }
      },
      series: [{
          name: '天然气用量',
          type: 'line',
          smooth: true,
          data: data,
          itemStyle: {
              color: '#2A9D8F'
          }
      }]
  };
  chart.setOption(option);
  return chart;
},



initEnergyCostPieChart: function (canvas, width, height, dpr, data) {
  const chart = echarts.init(canvas, null, {
    width,
    height,
    devicePixelRatio: dpr
  });
  canvas.setChart(chart);

  const total = data.reduce((sum, item) => sum + (item.value || 0), 0).toFixed(2);

  const option = {
    title: [
      {
        text: '能耗费用占比',
        left: 'center',
        top: 10,
        textStyle: {
          fontSize: 14
        }
      },
      {
        text: '总费用',
        subtext: `${total} 万元`,
        left: 'center',
        top: '45%',
        textStyle: {
          fontSize: 14,
          color: '#000'
        },
        subtextStyle: {
          fontSize: 14,
          color: '#000'
        }
      }
    ],
    tooltip: {
      trigger: 'item',
      formatter: '{b}: {c} 万元 ({d}%)'
    },
    legend: {
      orient: 'horizontal',
      bottom: 1,
      data: ['水费', '电费', '燃气费']
    },
    series: [{
      name: '能耗费用',
      type: 'pie',
      radius: ['40%', '60%'],
      center: ['50%', '55%'],
      data: data,
      label: {
        formatter: '{b}\n{d}%',
        fontSize: 12
      }
    }]
  };

  chart.setOption(option);
  return chart;
}



});
