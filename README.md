<!DOCTYPE html>
<html>
    <head>
        <link rel="preconnect" href="https://fonts.googleapis.com">
        <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
        <link href="https://fonts.googleapis.com/css2?family=Google+Sans:ital,opsz,wght@0,17..18,400..700;1,17..18,400..700&family=Pixelify+Sans:wght@400..700&display=swap" rel="stylesheet">
        <style>
            .txt {
                font-family: Google Sans;
                font-size: 20pt;  
            }
        </style>
    </head>
    <body>
        <div id="elapsed-label" class="txt" style="font-size: 12pt; text-align: center;">It is currently</div>
        <div id="elapsed" class="txt" style="text-align: center;"></div>
        <div class="txt" style="font-size: 12pt; text-align: center;">after Kirk</div>
        <div style="height: 70px;"></div>
        <label class="txt" style="display: block; font-size: 12pt; text-align: center;">
            Set end date and time:
            <input id="end-time" type="datetime-local" value="2026-10-07T00:00">
        </label>
        <label class="txt" style="display: block; font-size: 12pt; text-align: center;">
            <input id="days-toggle" type="checkbox">
            Show total days instead
        </label>
        <label class="txt" style="display: block; font-size: 12pt; text-align: center;">
            <input id="end-time-toggle" type="checkbox">
            Use configured end time
        </label>
        <script>
            const options = {
                timeZone: 'Europe/Budapest',
                year: 'numeric',
                month: 'numeric',
                day: 'numeric',
                hour: 'numeric',
                minute: 'numeric',
                second: 'numeric',
            };
            const formatter = new Intl.DateTimeFormat([], options);

            // Set the starting date and time here. Month is 1-12.
            const START_YEAR = 2025;
            const START_MONTH = 9;
            const START_DAY = 10;
            const START_HOUR = 18;
            const START_MINUTE = 23;
            const START_SECOND = 16;

            const startDate = new Date(
                START_YEAR,
                START_MONTH - 1,
                START_DAY,
                START_HOUR,
                START_MINUTE,
                START_SECOND,
            );

            // Month is 1-12; remaining date/time values use their usual ranges.
            function formatDate(year, month, day, hour = 0, minute = 0, second = 0) {
                const date = new Date(year, month - 1, day, hour, minute, second);
                return formatter.format(date);
            }

            function addCalendarMonths(date, months) {
                const result = new Date(date);
                const targetMonth = result.getMonth() + months;
                const targetYear = result.getFullYear() + Math.floor(targetMonth / 12);
                const normalizedMonth = ((targetMonth % 12) + 12) % 12;
                const day = Math.min(
                    result.getDate(),
                    new Date(targetYear, normalizedMonth + 1, 0).getDate(),
                );
                result.setFullYear(targetYear, normalizedMonth, day);
                return result;
            }

            function updateElapsedTime() {
                const elapsedElement = document.getElementById('elapsed');
                const showTotalDays = document.getElementById('days-toggle').checked;
                const useConfiguredEndTime = document.getElementById('end-time-toggle').checked;
                const endTimeValue = document.getElementById('end-time').value;
                const endDate = useConfiguredEndTime ? new Date(endTimeValue) : new Date();
                document.getElementById('elapsed-label').textContent = useConfiguredEndTime
                    ? 'Elapsed time until the configured time:'
                    : 'It is currently';

                if (useConfiguredEndTime && Number.isNaN(endDate.getTime())) {
                    elapsedElement.textContent = 'Choose a valid end date and time.';
                    return;
                }

                if (endDate < startDate) {
                    elapsedElement.textContent = 'The end time is earlier than the start time.';
                    return;
                }

                let cursor = new Date(startDate);
                let years = 0;
                while (addCalendarMonths(cursor, 12) <= endDate) {
                    cursor = addCalendarMonths(cursor, 12);
                    years++;
                }

                let months = 0;
                while (addCalendarMonths(cursor, 1) <= endDate) {
                    cursor = addCalendarMonths(cursor, 1);
                    months++;
                }

                let days = 0;
                while (true) {
                    const nextDay = new Date(cursor);
                    nextDay.setDate(nextDay.getDate() + 1);
                    if (nextDay > endDate) break;
                    cursor = nextDay;
                    days++;
                }

                const elapsedMilliseconds = showTotalDays
                    ? endDate.getTime() - startDate.getTime()
                    : endDate.getTime() - cursor.getTime();
                const totalSeconds = Math.floor(elapsedMilliseconds / 1000);
                const totalDays = Math.floor(totalSeconds / 86400);
                const hours = Math.floor((totalSeconds % 86400) / 3600);
                const minutes = Math.floor((totalSeconds % 3600) / 60);
                const seconds = totalSeconds % 60;

                elapsedElement.textContent = showTotalDays
                    ? `${totalDays} d, ${hours} h, ${minutes} m, ${seconds} s`
                    : `${years} y, ${months} mo, ${days} d, ${hours} h, ${minutes} m, ${seconds} s`;
            }

            document.getElementById('days-toggle').addEventListener('change', updateElapsedTime);
            document.getElementById('end-time-toggle').addEventListener('change', updateElapsedTime);
            document.getElementById('end-time').addEventListener('input', updateElapsedTime);
            updateElapsedTime();
            setInterval(updateElapsedTime, 1000);
        </script>
    </body>
</html>
