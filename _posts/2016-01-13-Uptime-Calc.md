---
title: Uptime Calculator
updated: 2017-01-26 20:00
---


<b>Enter the uptime SLA from your provider then hit enter.</b>
<h3>SLA: <input type='text' id='uptime' value='99.99' />%</h3>        
<table class='force-big'>
	<tr><th>Daily</th><th>Weekly</th><th>Monthly</th><th>Yearly</th></tr>
    <tr><td id='day'></td>    <td id='week'></td>    <td id='month'></td>    <td id='year'></td></tr>
</table>

<script src="http://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha256-3edrmyuQ0w65f8gfBsqowzjJe2iM6n0nKciPUp8y+7E=" crossorigin="anonymous"></script>
<script>
    calc_uptime(99.99);
    $(document).keydown(function(e) {
        if(e.which == 13) {
            calc_uptime($('#uptime').val());
        }
    });

    function calc_uptime($uptime) {
        var downtime_per_day_in_seconds=86400*((100-$uptime)/100); // this is off by a fraction of a second.
        var downtime_per_week_in_seconds=downtime_per_day_in_seconds*7; // this is off by seconds
        var downtime_per_month_in_seconds=downtime_per_day_in_seconds*30; // this assumes 30 day months with obvious issues
        var downtime_per_year_in_seconds=downtime_per_day_in_seconds*365; // this is off by .25 days due to the leap year            
        
        $('#day').html(human_readable_duration(downtime_per_day_in_seconds));
        $('#week').html(human_readable_duration(downtime_per_week_in_seconds));
        $('#month').html(human_readable_duration(downtime_per_month_in_seconds));
        $('#year').html(human_readable_duration(downtime_per_year_in_seconds));
    }

    function human_readable_duration(s) {            
        var days = Math.floor((s % 31536000) / 86400); 
        var hrs = Math.floor(((s % 31536000) % 86400) / 3600);
        var mins = Math.floor((((s % 31536000) % 86400) % 3600) / 60);
        var secs = (((s % 31536000) % 86400) % 3600) % 60;

        if (s<60)
            return round(secs) + "s";            
        else if (s<3600)
            return round(mins) + "m " + round(secs) + "s";            
        else if(s<86400)
            return round(hrs) + "h " + round(mins) + "m " + round(secs) + "s";            

        return round(days) + "d " + round(hrs) + "h " + round(mins) + "m " + round(secs) + "s";            
    }

    function round(x) {
        if(x < 0.001) // anything that small might as well be 0.001.
            return 0.001;
        else
            return Number.parseFloat(x).toPrecision(3);
    }
</script>    
