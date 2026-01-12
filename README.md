## caldav.h - Header-only C/C++ CalDAV client library

A minimal CalDAV client for C and C++ projects.
The only dependency is `libcurl` for making HTTP requests.

Features:

- **Calendar Management**:
  - Automatic discovery via `/.well-known/caldav/`
  - Fetch calendars with properties: Name, Description, Color
  - Create, update, and delete calendars
  - Change tracking with CTags
- **Event Management**:
  - Get events with iCalendar data (RFC 5545)
  - Create, update, and delete events
  - Change tracking with ETags
- **Simple API**: Clean, consistent interface with full documentation
- **Header-Only**: Single [stb-style](https://github.com/nothings/stb) header file that you can put it directly in your project.

### Usage

```c
#define CALDAV_DEBUG          // Define for debug logs
#define CALDAV_IMPLEMENTATION // Define in **ONE** source file to include the implementation
#include "caldav.h"

#include <stdio.h>

#define USERNAME "vlad"
#define PASSWORD "1234"
#define URL      "localhost:8080" // Nextcloud
// #define URL      "localhost:5232" // Radicale

// Event iCal data
#define EXAMPLE_EVENT_UID "123e4567-e89b-12d3-a456-426614174010"
const char *example_event = "BEGIN:VCALENDAR\n"
                            "VERSION:2.0\n"
                            "PRODID:-//Example//CalDAV Test//EN\n"
                            "BEGIN:VTODO\n"
                            "UID:" EXAMPLE_EVENT_UID "\n"
                            "DTSTAMP:20260107T120000Z\n"
                            "SUMMARY:Test event created via CalDAV\n"
                            "DESCRIPTION:Created via CalDAV\n"
                            "END:VTODO\n"
                            "END:VCALENDAR\n";

int main() {
  // Connect to the server
  CalDAVClient *client = caldav_client_new(URL, USERNAME, PASSWORD);
  if (!client) {
    fprintf(stderr, "Failed to create client\n");
    return 1;
  }
  // Pull calendars from the server
  caldav_client_pull_calendars(client);
  // Create new VTODO calendar
  bool created = caldav_client_create_calendar(client, "my-calendar-uuid", "Calendar Name", "My calendar description",
                                               "#181818ff", CALDAV_COMPONENT_SET_VTODO);
  if (!created) {
    fprintf(stderr, "Failed to create calendar\n");
    return 1;
  }
  // Loop calendars
  for (size_t i = 0; i < client->calendars->count; ++i) {
    // Get calendar from calendar list
    CalDAVCalendar *calendar = client->calendars->items[i];
    // Check only VTODO calendars
    if (!(calendar->component_set & CALDAV_COMPONENT_SET_VTODO)) continue;
    // Print calendar information
    caldav_calendar_print(calendar);
    // Update calendar properties
    bool updated = caldav_request_calendar_update(calendar, "Changed Name", "Changed description", "#FFFFFFFF");
    if (!updated) {
      fprintf(stderr, "Failed to update calendar\n");
      return 1;
    }
    // Print new calendar information
    caldav_calendar_print(calendar);
    // Pull calendar events from the server
    caldav_calendar_pull_events(client->calendars->items[i]);
    // Create new event
    caldav_calendar_create_event(client->calendars->items[i], EXAMPLE_EVENT_UID, example_event);
    // Print all events
    for (size_t j = 0; j < client->calendars->items[i]->events->count; ++j)
      caldav_event_print(client->calendars->items[i]->events->items[j]);
    // Delete first event
    CalDAVEvent *first_event = calendar->events->items[0];
    caldav_event_delete(first_event);
    if (!first_event->deleted) {
      fprintf(stderr, "Failed to delete event\n");
      return 1;
    }
  }
  // Delete last calendar
  bool deleted = caldav_calendar_delete(client->calendars->items[client->calendars->count - 1]);
  if (!deleted) {
    fprintf(stderr, "Failed to delete calendar\n");
    return 1;
  }
  // Cleanup client
  caldav_client_free(client);

  return 0;
}
```

### Documentation

Look inside `caldav.h`. All functions have comments.

### Building and Testing

To build the shared library, run the following command:

```sh
gcc -x c -shared -o libcaldavh.so caldav.h
```

To run the example, run the following command:

```sh
gcc -Wall -Wextra -Wpedantic -o example example.c -lcurl -g && ./example
```

Make sure to change the URL, USERNAME and PASSWORD in `example.c` to your own CalDAV server.

### Contributing

Pull requests are welcome!

### License

This project is licensed under the Zlib license - see the [LICENSE](LICENSE) file for details.

### TODO

- [x] Autodiscovery of CalDAV URL
- [x] Calendars
  - [x] Getting properties
    - [x] URL
    - [x] CTag
    - [x] Name
    - [x] Description
    - [x] Color
  - [x] Updating
    - [x] Fetch created
    - [x] Fetch changed
    - [x] Fetch deleted
    - [x] Push properties
      - [x] Name
      - [x] Description
      - [x] Color
  - [x] Deleting
- [x] Events
  - [x] Getting
    - [x] URL
    - [x] ETag
    - [x] iCal
  - [x] Updating
    - [x] Fetch created
    - [x] Fetch changed
    - [x] Fetch deleted
    - [x] Push new ical
  - [x] Deleting
