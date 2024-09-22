# Blue Marble Weather
Blue Marble Weather is a tvOS App built using Swift and SwiftUI that integrates Apple's Apple Weather API and the OpenDataSoft Geoname locations database. You can search for cities/towns (with a population > 1000) to find the current, daily and hourly weather data. 

Blue Marble doesn't link your identity to your current location. It only saves your recently searched locations and provides custom settings for temperature and wind speed unit conversions, as well as custom theme color settings.

# Technologies Used
* SwiftUI
* Apple Weather
* Unit Testing
* DocC documentation
* Git & GitHub

# Animated Weather Data
<figure>
	<div>
		<video width="500" controls poster="../Images/BlueMarbleWeather/BMW MainView Daily.png" muted preload="auto">
			<source src="../Videos/BlueMarbleWeather_compressed.mp4" type="video/mp4">
			<!-- For non-HTML5 browsers: -->
			Your browser doesn't support the video tag. Click <a href=http://www.firefox.com>here</a> 
			to download the Firefox browser for your operating system.
		</video>
	</div>
</figure>

# SwiftUI vs UIKit...
It's definitely a big difference working in SwiftUI, as compared to UIKit. Using SwiftUI mechanisms like observable objects, @State and @Binding, for passing around data and updating views, is definitely a lot faster to implement as compared to implementing delegates and building out UIKit components, such as, table views.

One of the things I enjoyed learning was how to integrate data from the Apple Weather API in a scrolling horizontal list view showing daily or hourly weather data, and to be present additional weather data below the view while selecting individual list items.

Below is the code for retrieving current, daily and hourly weather:

```swift
import Foundation
import WeatherKit
import CoreLocation

/// Includes the 'service' WeatherKit WeatherSearch.shared object.
///
/// Will primarily use the 'shared' static let property to access WeatherData methods.
class WeatherData: ObservableObject {
    static let shared   = WeatherData()
    private let service = WeatherService.shared
    
    func currentHourlyDailyForecast(for location: CLLocation) async -> (CurrentWeather, Forecast<HourWeather>, Forecast<DayWeather>)? {
        let currentDailyHourlyForcast = await Task.detached(priority: .userInitiated) {
            let forcast = try? await self.service.weather(
                for: location,
                including: .current, .hourly, .daily)
            return forcast
        }.value
        return currentDailyHourlyForcast
    }
}
```
<br>
</br>

The daily/hourly scrolling view, called from the main view:

```swift
switch forecastSelection {
                case .currently:
                    CurrentlyView(currentWeather: currentWeather)
                case .daily:
                    HScrollViewForecast(weather: $dailyWeather, selectedItem: $selectedItem)
                        .onChange(of: selectedItem) { oldValue, newValue in
                            if let dailyWeather = dailyWeather, let selectedItem = selectedItem, (0..<dailyWeather.count).contains(selectedItem) {
                                selectedDayWeather = dailyWeather[selectedItem]
                            }
                            
                        }
                case .hourly:
                    HScrollViewForecast(weather: $hourlyWeather,
                                        selectedItem: $selectedItem)
                    .onChange(of: selectedItem) { oldValue, newValue in
                        if let hourlyWeather = hourlyWeather, let selectedItem = selectedItem, (0..<hourlyWeather.count).contains(selectedItem) {
                            selectedHourWeather = hourlyWeather[selectedItem]
                        }
                    }
                }
```
<br>
</br>


Code for the HScrollViewForecast:

```swift
/// The horizontal scroll view that presents the hourly or daily forcast.
/// - Note: Both Forecast<HourWeather> and Forecast<DayWeather> conform to the RandomAccessCollection Protocol.
struct HScrollViewForecast<T: RandomAccessCollection>: View {
    @Binding var weather: T?
    
    @FocusState.Binding var selectedItem: Int?
    
    var body: some View {
        HStack {
            ScrollView(.horizontal) {
                HStack(spacing: 20) {
                    switch weather {
                    case is Forecast<HourWeather>:
                        if let weather = weather {
                            ForEach(Array(weather.enumerated()), id: \.offset) { index, weather in
                                HourlyView(hourWeather: weather as? HourWeather,
                                          selectedItem: $selectedItem,
                                          index: index)
                            }
                            .padding()
                        }
                    // By default, show DayWeather:
                    default:
                        if let forecast = weather {
                            ForEach(Array(forecast.enumerated()), id: \.offset) { index, weather in
                                DailyView(dayWeather: weather as? DayWeather,
                                          selectedItem: $selectedItem,
                                          index: index)
                            }
                            .padding()
                        }
                    }
                }
            }
            .frame(width: 1000)
        }
    }
}
```
<br>
</br>


Code for loading weather data from the main view:

```swift
.task {
                isLoadingData = true
                
                ...
                
                // Gets Initial GeoLocation Weather Data:
                Task.detached {
                    if let geoLocation = await geoFavoritesVM.geoLocationFrom(favorite: favoriteLocation) {
                        await getWeatherData(for: geoLocation.cllLocation)
                    }
                }
                
				...
            }
            
            ...
            
            func getWeatherData(for location: CLLocation) async {
        Task.detached {
            if let (current, hourly, daily) = await weatherServiceHelper.currentHourlyDailyForecast(for: location) {
                let weatherAttribution      = await weatherServiceHelper.weatherAttribution()
                
                await MainActor.run {
                    attribution             = weatherAttribution
                    isLoadingData           = false
                    self.currentWeather     = current
                    self.dailyWeather       = daily
                    self.hourlyWeather      = hourly
                }
            }
        }
    }
```
<br>
</br>

<table>
<tr>
	<td>
	<img src="../images/BlueMarbleWeather/BMW MainView Daily.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
	<td>
	<img src="../images/BlueMarbleWeather/BMW Search Screen.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
</tr>
<tr>
	<td>
	<img src="../images/BlueMarbleWeather/BMW Info Screen.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
	<td>
	<img src="../images/BlueMarbleWeather/BMW Settings Screen.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
</tr>
</table>

<!-- ![readme-bluemarbleweather-daily](../images/BMW%20MainView%20Daily.png) -->

# Code Implementations
Although it's a simple project, I've implemented the following:
* MVVM Design Pattern; using ObservableObjects, Models and Views.
* SwiftUI components: ObservableObject, @EnvironmentObject, @ObservableObject, @StateObject, @State, @Binding
* Unit Testing
* Code documentation, DocC
* Error handling
* Alerts
* Empty states
* Text input validation
* Project organization

# Future Considerations
* Implement SwiftData, for storing larger amounts of location data
* Add a TabView for showing Weather Trends in Charts
* Implement a Weather Radar API
* Add Accessibility
* Add Localizations
* Add iOS, macOS, and VisionOS compatibility
