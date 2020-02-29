---
layout: post
title:      "Visit More Parks React and Rails App"
date:       2020-02-29 05:28:21 +0000
permalink:  visit_more_parks_react_and_rails_app
---

- Draft -

Wouldn't it be great if you could see all nearby United States national parks when you're reviewing upcoming travel plans? My [React](https://github.com/aparkening/visit-more-parks-frontend) and [Rails](https://github.com/aparkening/visit-more-parks-backend) student project, _Visit More Parks_, scans your upcoming Google Calendar events, suggests parks that are within 100 miles of each destination, and allows you to easily create a calendar event to visit a specific park, including relevant park information as calendar data. You can also browse all National Park Service parks and highlight parks to visit in the future.
 
## App Features
- Users log in with their Google Calendar account.
- All U.S. National Parks are pre-loaded with geocoded coordinates and addresses using the nps.gov parks API and the Geocoder Ruby gem.
- Each upcoming event with a United States location are automatically matched with parks that are within 100 miles.
- Events created in the app automatically includes park information, such as description, address, and URL.
- Events that are created, modified, and deleted within the app automatically synchronize with Google Calendar.
- Users can browse all parks and favorite parks they wish to visit in the future.

## Why
As a final student project, I wanted to pull in real-world complexity by interacting with multiple data sources, using Oauth to authenticate with an external provider, and actually write and update resources on another service.

The National Park Service has an HTTP-based API that returns robust, but straightforward JSON data. The difficult part came when interacting with Google's Calendar API, a much more complex data stream that requires a persistent authorization token and has several methods for interacting with calendars and events. 

## Not the Easy One
The latest, fastest, and most supported, method of using Google's Calendar API is with Google's very robust [JavaScript browser client](https://github.com/google/google-api-javascript-client), also known as `gapi`. But! My project required 1) a Rails backend that persisted data, and 2) a React frontend that displayed data with minimal data manipulation. Using JavaScript as middleware to grab Google Calendar data, process it, and send along to the Rails server didn't make much sense with those project requirements, so I investigated further. 

Luckily, Google also created a [Ruby API client](https://github.com/googleapis/google-api-ruby-client). It won't receive any new features, but it's still officially maintained and has some documentation. The good news is that I could incorporate a Ruby on Rails-native client into my project! The bad news is that the documentation was written for _all_ Google services, so it was difficult to glean what applied to my situation, it didn't clearly illustrate how to securely deal with authenticating credentials between requests, and the examples were confusingly different between the [GitHub examples](https://github.com/googleapis/google-api-ruby-client/blob/master/samples/cli/lib/samples/calendar.rb) and the official website [API reference examples](https://developers.google.com/calendar/v3/reference/events/insert). When confused about how to interact with the ruby client, I dove directly into the [service file](https://github.com/googleapis/google-api-ruby-client/blob/b9e9a8e2b8dd45d0ad3cd7937a8cb39ea52f9c36/generated/google/apis/calendar_v3/service.rb#L1174) to see how each method was created and documented.

## Solution

I first tackled Oauth and Google by reading many tutorials. A couple handy articles were [_Google OAuth for Ruby On Rails_](https://medium.com/@amoschoo/google-oauth-for-ruby-on-rails-129ce7196f35) and [_Using The Google API Ruby Client with Google Calendar API_](https://www.thegreatcodeadventure.com/using-the-google-api-ruby-client-with-google-calendar-api/).

Most articles used Devise to handle of authorization details, but I wanted a more light-weight solution, so I built my own re-usable methods for reading, creating, updating, and destroying events.

```
require 'google/api_client/client_secrets.rb'
require 'google/apis/calendar_v3'

class Api::V1::EventsController < ApplicationController
  before_action :authenticate

  # All events
  def index
    # Only keep locations that have comma (indicates city, state)
    location_hash = get_google_events["items"].select{|event| event["location"] && event["location"].include?(",")}

    ### Sample output
    # location_names = location_hash.each{|e| puts e["summary"] +" - "+ e["location"]}
    # Output: 
    # Boston Trip - Boston, MA
    # Minneapolis Trip - Minneapolis, MN
    # Austin Trip - Austin, TX
    # Portland Trip - Portland, OR
    # Virginia Trip - Falls Church, VA

    ### Sample geocode usage
    # g = Geocoder.search("Boston, MA")
    # g.first.coordinates #=> [42.3602534, -71.0582912]

    # Add array of parks within 100 miles to each location
    events_and_parks = location_hash.each{|event| event["nearParks"]= Park.near(event["location"], 100).as_json}

    # Return events with parks
    render json: { events: events_and_parks }
  end


  # Display record
  def show
    event = Event.find_by(id: params[:id])

    if event
      # Render json
      render json: { event: event }
    else
      not_found
    end
  end


  # Create record
  def create
    event = current_user.events.build(event_params)

    # If event can save, also send to Google Calendar
    if event.save

      # Start Google calendar
      calendar = start_google_service

      # Format event for Google
      g_event = Google::Apis::CalendarV3::Event.new(
        summary: event.title,
        location: event.location,
        description: event.description,
        start: Google::Apis::CalendarV3::EventDateTime.new(
          date_time: event.start_time,
          time_zone: event.timezone
        ),
        end: Google::Apis::CalendarV3::EventDateTime.new(
          date_time: event.end_time,
          time_zone: event.timezone
        )
      )
			
			# Add Google Calendar event
      result = calendar.insert_event('primary', g_event)

			# Add Google Calendar id to event table
      event.g_cal_id = result.id
      event.save

      # Render json
      render json: {event: event}
    else
      resource_error
    end
  end

  # Update record
  def update
    event = Event.find(params[:id])
    event.update(event_params)
    
    # If event can save, also send to Google Calendar
    if event.save

      # Start Google calendar
      calendar = start_google_service

      # Format event for Google
      g_event = format_google_event(event)

			# Update Google Calendar event
      result = calendar.update_event('primary', event.g_cal_id, g_event)

      # Return event id
      render json: {event: event}
    else
      resource_error
    end
  end

  # Delete record
  def destroy
    event = Event.find(params[:id])
    
    # Only event owner can delete
    authorize_resource(event)
    
    # Start Google calendar
    calendar = start_google_service

    # Delete Google event
    result = calendar.delete_event('primary', event.g_cal_id)

    # Delete database event
    event.destroy

    # Render json
    render json: { event: event.id}
  end

  private

  def event_params
		params.require(:event).permit(:title, :location, :description, :start_time, :end_time, :timezone, :user_id, :park_id)
  end

  # Start calendar service and authorize use
  def start_google_service
    # Initialize Google Calendar API
    service = Google::Apis::CalendarV3::CalendarService.new
    # Use google keys to authorize
    service.authorization = google_secret.to_authorization
    # Request new access token in case it expired
    service.authorization.refresh!

    return service
  end

  # Tokens and client env variables
  def google_secret
    Google::APIClient::ClientSecrets.new(
      { "web" =>
        { "access_token" => current_user.google_token,
          "refresh_token" => current_user.google_refresh_token,
          "client_id" => ENV['GOOGLE_CLIENT_ID'],
          "client_secret" => ENV['GOOGLE_CLIENT_SECRET']
        }
      }
    )
  end

  # Return hash of google events
  def get_google_events
    calendar = start_google_service
    
    # Set calendar
    # Primary is main account
    calendar_id = "primary"

    # Get up to 1000 events in calendar
    events = calendar.list_events(
      calendar_id,
      max_results: 1000,
      single_events: true,
      order_by: "startTime",
      time_min: DateTime.now.rfc3339
    )
    
    # Convert event list into hash
    return JSON.parse(events.to_json)
  end
end
```
