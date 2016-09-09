Time Zones
==========

##System Time
```ruby
Date.today
DateTime.now
Time.now
```
This gives the users system time, or the application time, if run from cron etc.

##Explicit Time Zone
```ruby
Date.current
DateTime.current
Time.current
Time.zone...
2.hours...
3.days...
Time.[METHOD].in_time_zone
```
This ensures we are working with a specific time zone

##Setting the time zone to set the users time zone
```ruby
# app/controllers/application_controller.rb
around_action :set_time_zone, if: :current_user

private

def set_time_zone(&block)
  Time.use_zone(current_user.time_zone, &block)
end
```
See more here: [It's About Time
(Zones)](https://robots.thoughtbot.com/its-about-time-zones)

##Form Helpers
```ruby
f.input :time_zone, priority: /US/
```
Simple form helper, more [here](https://github.com/plataformatec/simple_form)
This uses [ActiveSupport::TimeZone <
Object](http://api.rubyonrails.org/classes/ActiveSupport/TimeZone.html)

This simplifies the time zones, by reducing them, e.g: 'Canberra' is mapped to
'Australia/Melbourne' several others are skipped when there is a favourable
duplicate.

Determining the users time zone. The browser time zone needs to be detected via
JS. EG: [browser_timezone_rails](https://github.com/kbaum/browser-timezone-rails)

Care needs to be taken when pulling a users timezone from js, as it won't be
recognised by the form helper.

```ruby
var tz = jstz.determine();
tz.name();
'Europe/Berlin'
```
This time zone is mapped to 'Berlin', so won't appear pre-selected, even though
it will return the same TZInfo object through the find_tzinfo method
```ruby
# File activesupport/lib/active_support/values/time_zone.rb, line 202
def find_tzinfo(name)
  TZInfo::Timezone.new(MAPPING[name] || name)
end
```
We could do the following when saving the name value:
```ruby
def get_time_zone
  z = cookies[:timezone]
  if ActiveSupport::TimeZone::MAPPING.has_key? z
    return z
  elsif ActiveSupport::TimeZone::MAPPING.has_value? z
    return ActiveSupport::TimeZone::MAPPING.key(z)
  else
    return ????
  end
end
```
This solves the above problem, though many zones will be skipped.
Also, we have the problem where "Canberra" is mapped to "Australia/Melbourne",
not "Australia/Canberra"

Options:
1. Find the time zone info, determine the offset and then find a zone which is
   safe for the form helpers:
```bash
c_info = ActiveSupport::TimeZone.find_tzinfo("Australia/Canberra")
=> #<TZInfo::TimezoneProxy: Australia/Canberra>
offset = c_info.utc_to_local(0)
=> 36000
safe_zone = ActiveSupport::TimeZone[offset]
=> #<ActiveSupport::TimeZone:0x007fcc8cce0ad0
 @current_period=#<TZInfo::TimezonePeriod:
#<TZInfo::TimezoneTransitionDefinition: #<TZInfo::TimeOrDateTime:
#699379200>,#<TZInfo::TimezoneOffset: 36000,0,AEST>>,nil>,
#@name="Brisbane",
#@tzinfo=#<TZInfo::TimezoneProxy: Australia/Brisbane>,
#@utc_offset=nil>
```
This unfortunately (at least in this case) returns Brisbane, as it's the first
match for the given offset. This is problematic, as brisbane uses AEST all year and Canberra uses AEST in the winter and AEDT (daylight savings) in the summer. Also, 'America/New_York' would be
mapped to 'America/Bogota', which is probably not what users want to see.

```bash
# File activesupport/lib/active_support/values/time_zone.rb, line 227
def [](arg)
  case arg
    when String
    begin
      @lazy_zones_map[arg] ||= create(arg)
    rescue TZInfo::InvalidTimezoneIdentifier
      nil
    end
    when Numeric, ActiveSupport::Duration
      arg *= 3600 if arg.abs <= 13
      all.find { |z| z.utc_offset == arg.to_i }
    else
      raise ArgumentError, "invalid argument to TimeZone[]: #{arg.inspect}"
  end
end
```

So we could just change that method to user `find_all` and return all the
possible zones, then try to find the best match on the name.

```bash
ActiveSupport::TimeZone.all.find_all { |z| z.utc_offset == offset }
[#<ActiveSupport::TimeZone:0x007fcc8cce0ad0
  @current_period=#<TZInfo::TimezonePeriod:
#<TZInfo::TimezoneTransitionDefinition: #<TZInfo::TimeOrDateTime:
699379200>,#<TZInfo::TimezoneOffset: 36000,0,AEST>>,nil>,
  @name="Brisbane",
  @tzinfo=#<TZInfo::TimezoneProxy: Australia/Brisbane>,
  @utc_offset=nil>,
 #<ActiveSupport::TimeZone:0x007fcc8bf5e088
  @current_period=
   #<TZInfo::TimezonePeriod: #<TZInfo::TimezoneTransitionDefinition:
#<TZInfo::TimeOrDateTime: 1459612800>,#<TZInfo::TimezoneOffset:
36000,0,AEST>>,#<TZInfo::TimezoneTransitionDefinition: #<TZInfo::TimeOrDateTime:
1475337600>,#<TZInfo::TimezoneOffset: 36000,3600,AEDT>>>,
  @name="Canberra",
  @tzinfo=#<TZInfo::TimezoneProxy: Australia/Melbourne>,
  @utc_offset=nil>,
etc etc
```

Then we could test the name against our js value using a regular expression.
But that wouldn't work for "America/Vancouver", which maps to "Pacific Time (US & Canada)"

2. Static mappings using [these
   values](https://gist.github.com/jpmckinney/767070)

Simpler :)
