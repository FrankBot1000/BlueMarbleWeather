# Blue Marble Weather
<p>
Blue Marble Weather is a tvOS App, built using Swift and SwiftUI, for showing the current, daily, and hourly weather forecast via Apple's Weather API,
<a href="https://developer.apple.com/weatherkit/data-source-attribution/"> 
		<img width="80" src="images/pics/Apple_Weather_wht_en_3X_090122.png" alt="Apple Weather Image"</img> 
</a>
<div style="display: inline-block;">
	<a href="https://developer.apple.com/weatherkit/data-source-attribution/"> 
		<img width="80" src="images/pics/Apple_Weather_blk_en_3X_090122.png" alt="Apple Weather Image"</img> 
	</a>
</div>

This App doesn't link your identity to your current location. It only saves your recently searched locations and provides custom settings for temperature and measurement system conversions, as well as custom theme color settings. (The North Pole is the default location.)
</p>

<h2>Available for AppleTV</h2>
<a href="https://apps.apple.com/app/blue-marble-weather/id6642685383">
	<img src="images/pics/Download_on_Apple_TV_Badge_US-UK_RGB_blk_092917.svg" alt="Download on Apple TV"</img> 
</a>
<br></br>

# Technologies Used
* SwiftUI
* Apple Weather
* Unit Testing
* DocC Documentation
* Git & GitHub
<br></br>


# Animated Weather Data

[//]: # "NB: For README.md Github videos, Use GitHub asset urls eg. https://github.com/user-attachments/assets/xxxxxPlaceholderFileNameHerexxxxx as video source (derived first by dragging-dropping a video within the README.md file to get the url)."
<video width="500" src="https://github.com/user-attachments/assets/9af2a3d1-b039-4f58-b1ca-05de93df3089">
</video>


[//]: # "For webpage, use embedded below figure instead."
 <!--
<figure>
	<div>
		<video width="500" controls poster="images/screenshots/BMW MainView Daily.png" muted preload="auto">
			<source src="videos/BlueMarbleWeather_compressed.mp4" type="video/mp4"></source>
			<!- - For non-HTML5 browsers: - ->
			Your browser doesn't support the video tag. Click <a href=http://www.firefox.com>here</a> 
			to download the Firefox browser for your operating system.
		</video>
	</div></figure>
 </figure>
 -->

<br></br>


# SwiftUI vs UIKit...
It's definitely a big difference working in SwiftUI, as compared to UIKit. Using observable objects, @State and @Binding, for passing around data and updating views, is definitely a lot faster to implement as compared to implementing delegates and building out UIKit components, such as, collection views and table views.

One of the things I enjoyed learning was how to manage data from the Apple Weather API in a scrolling horizontal list view, that showed daily or hourly weather data, and presenting additional weather data below the view when selecting individual list items.
<br></br>


## Below are Some Code Examples:
### Retrieving current, daily and hourly weather:
```swift
func currentHourlyDailyForecast(for location: CLLocation) async -> (CurrentWeather, Forecast<HourWeather>, Forecast<DayWeather>)? {
	let currentDailyHourlyForcast = await Task.detached(priority: .userInitiated) {
		let forcast = try? await self.service.weather(
			for: location,
			including: .current, .hourly, .daily)
		return forcast
	}.value
	return currentDailyHourlyForcast
}
```
<br></br>


### The daily/hourly scrolling view, called from the Main View:
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
<br></br>


### Code for the horizontal ScrollView Forecast:
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
<br></br>


### Code for loading weather data from the Main View:
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
<br></br>


# Sample Screen Shot Images
<table>
<tr>
	<td>
	<img src="images/screenshots/02 BMW Home Daily North Pole 041425.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
	<td>
	<img src="images/screenshots/05 BMW Search Indianapolis 041425.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
</tr>
<tr>
	<td>
	<img src="images/screenshots/06 BMW InfoView Blue 052325 V152B657.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
	<td>
	<img src="images/screenshots/06c BMW Settings Blue 041425.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
</tr>
</table>
<br></br>


## Although it's a smaller project, I've implemented the following:
#### Code Structure
* Model-View-View-Model (MVVM) Design Pattern
* SwiftUI components: 
	* ObservableObject, @EnvironmentObject, @ObservedObject, @StateObject
	* @State, @Binding, @FocusState, @Published
* Generics

#### Testing/Error Handling
* Unit Testing
* Error Handling
* Alerts
* Empty States
* Text Input Validation

#### Accessibility
* Light/Dark Mode Settings
* VoiceOver Accessibility (buttons & charts)

#### Customizations
* Light and Dark Mode Options
* Theme Color Options
* Unit Conversions
* Startup Screen Options

#### Project Organization
* Code Documentation (DocC)
* Group Folder Organization
* Privacy Manifest

# Future Considerations
* Implement SwiftData, for storing larger amounts of location data
* Add a TabView for showing Weather Trends in Charts
* Implement a Weather Radar API
* Add Accessibility, Localizations
* Add iOS, macOS, and VisionOS compatibility
<br></br>
<br></br>


# FeedBack
<p class="contact-message">If you have any feedback or suggestions you can reach out via <a class="btn" href="mailto:fbotlogic@fbotlogicsolutions.com?subject=Blue Marble Weather Support">Email</a>.</p>

