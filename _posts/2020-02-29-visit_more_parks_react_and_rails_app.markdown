---
layout: post
title:      "Visit More Parks React and Rails App"
date:       2020-02-29 00:28:22 -0500
permalink:  visit_more_parks_react_and_rails_app
---


Wouldn't it be great if you could see National Parks that are close to places you're already visiting in the US? My [React](https://github.com/aparkening/visit-more-parks-frontend) and [Rails](https://github.com/aparkening/visit-more-parks-backend) student project, _Visit More Parks_, scans your upcoming Google Calendar events, geocodes locations for those events, and suggests parks that are within 100 miles of each destination. Taking advantage of the suggestions, the app also makes it easy to add a park visit to your calendar.
 
## What It Does
- The user logs in with their Google Calendar account.
- US National Parks are pre-loaded with geocoded coordinates and addresses (using the [NPS.gov Parks API](https://www.nps.gov/subjects/developer/index.htm) and the [Geocoder Ruby gem](https://github.com/alexreisner/geocoder)).
- Upcoming calendar events are geocoded and displayed with their nearby parks.
- The user can add (as well as modify or delete) visits to any of the parks as calendar events.
- Those new events include park information, such as description, address, and URL.
- All 497 National Parks are browsable.

## Why
As a final student project, I wanted to try integrating and manipulating multiple data sources within a single cohesive app. I also wanted to allow users to interact with their own Google information outside of a Google-specific app and without logging in fresh every time.

The National Park Service has an HTTP-based API that returns robust, straightforward JSON data. Google Calendar has a complex API, involving persistent authorization tokens, multiple interaction methods, and variability within returned data. That's a lot to tackle! 

## Finding the Right Calendar API
The latest, fastest, and best-supported method of using Google's Calendar API is with Google's robust [JavaScript browser client](https://github.com/google/google-api-javascript-client), also known as `gapi`. The requirements for my project specified a Rails backend to persist data and a React frontend that _minimally_ manipulated data, so using JavaScript/React as middleware to manipulate Google Calendar data and then send the data along to a Rails server didn't make much sense. What I really wanted was a way for Rails to grab and process all the data and simply supply React with some easy JSON feeds.

Luckily, Google also created a [Ruby API client](https://github.com/googleapis/google-api-ruby-client). It's not as robustly documented as the Javascript browser client, but it _is_ still officially maintained for big fixes and does have _some_ documentation! 

## Persistent Authentication
My excitement quickly wore off when I realized the Ruby API and documentation applied to _all_ Google services. I found it difficult to glean what was relevant to my situation, especially regarding authentication credentials between requests. 

Combing through a ton of online resources, I found a couple articles that outlined some helpful approaches to refreshing authorization: [_Google OAuth for Ruby On Rails_](https://medium.com/@amoschoo/google-oauth-for-ruby-on-rails-129ce7196f35) and [_Using The Google API Ruby Client with Google Calendar API_](https://www.thegreatcodeadventure.com/using-the-google-api-ruby-client-with-google-calendar-api/). The key to successful authorization was to initialize a new Google Calendar each time my controller created, read, updated, or destroyed a calendar event. So I created some helper methods patterned after the calendar_list approach from _Google OAuth for Ruby On Rails_. 

```
  # Start and authorize new calendar service
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
```

## Interacting with Google Calendar
Once I understood how to authenticate each Google Calendar request, I needed to actually create, read, update, and destroy events. The official examples only covered a couple interaction scenarios and were also confusingly different between the [GitHub versions](https://github.com/googleapis/google-api-ruby-client/blob/master/samples/cli/lib/samples/calendar.rb) and the [website API versions](https://developers.google.com/calendar/v3/reference/events/insert). 

After unsuccessful attempts to expand the canned interaction examples, I dove directly into the API client code to understand the available methods and their expected parameters. The [service file](https://github.com/googleapis/google-api-ruby-client/blob/b9e9a8e2b8dd45d0ad3cd7937a8cb39ea52f9c36/generated/google/apis/calendar_v3/service.rb#L1174) provided the gold I needed: all the interaction methods _and_ code comments listing their required parameters!

Using my newfound knowledge, I was able to create additional helper methods to grab existing event lists and format Google-friendly events.

```  
  # Return hash of Google events
  def get_google_events
    calendar = start_google_service
    
    # Set calendar ('primary' is main Google account)
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
  
	# Return event hash in Google-friendly format  
	def format_google_event(event)
    Google::Apis::CalendarV3::Event.new(
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
  end
end
```

Using the authentication and events helpers, my index, create, update, and delete methods became nicely straightforward:
```
# events_controller.rb

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
    # location_name: Boston Trip - Boston, MA

    # Add array of parks within 100 miles to each location
    events_and_parks = location_hash.each{|event| event["nearParks"]= Park.near(event["location"], 100).as_json}

    # Return events with parks
    render json: { events: events_and_parks }
  end

  # Create record
  def create
    event = current_user.events.build(event_params)

    # If event can save to database, also send to Google Calendar
    if event.save

      # Start Google calendar
      calendar = start_google_service

      # Format event for Google
			g_event = format_google_event(event)
			
			# Add event to Google Calendar
      result = calendar.insert_event('primary', g_event)

			# Add Google Calendar id to database
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
  
  # helper methods

end
```

## Finding Answers in Articles and Code
Reading about other people's experiences and directly diving into the Ruby code was much more valuable than slogging through the confusing official Google Calendar API documentation. Where the documentation lacked, external resources illustrated robust approaches and gave me hope that my own app could work. And reading the API methods provided the needed interaction requirements while showing me the product wasn't nearly as complicated as it seemed. Now users can seamlessly interact with Google Calendar through my app!
