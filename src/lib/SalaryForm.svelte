<script>
    import {differenceInBusinessDays, getDaysInMonth} from 'date-fns'

    let salary = localStorage.getItem(SALARY_LOCAL_STORAGE_KEY) ?? 0;

    let daysInMonth;

    let hoursInMonth;
    let hoursOff;
    let hoursWorked;

    let holidays = 0;
    let offlineDays = 0;
    let offlineHours = 0;

    let alertMessage;
    let invoice = 0;
    let calculated = false;

    const SALARY_LOCAL_STORAGE_KEY = 'salary_value';
    const monthNames = ["January", "February", "March", "April", "May", "June",
        "July", "August", "September", "October", "November", "December"
    ];

    const currentDate = new Date();
    let month = currentDate.getMonth();

    function calculateDaysInMonth() {
        return differenceInBusinessDays(
            new Date(currentDate.getFullYear(), month, getDaysInMonth(new Date(currentDate.getFullYear(), month))),
            new Date(currentDate.getFullYear(), month, 1)
        );
    }

    $: if (month) {
        daysInMonth = calculateDaysInMonth();
    }

    function calculate() {
        calculated = false;
        alertMessage = '';
        if (holidays < 0 || holidays > daysInMonth) {
            alertMessage = 'Holidays cannot be greater than days in the month'
            return;
        }

        hoursInMonth = (daysInMonth - holidays) * 8;
        hoursWorked = (daysInMonth - holidays - offlineDays) * 8 - offlineHours;
        hoursOff = offlineDays * 8 + offlineHours;
        if (hoursOff > hoursInMonth) {
            alertMessage = `Offline hours (${hoursOff}) cannot be greater than hours in the month (${hoursInMonth})`;
            return;
        }

        invoice = Math.ceil((salary * hoursWorked) / hoursInMonth);
        calculated = true;
    };

    function storeSalary(ev) {
        localStorage.setItem(SALARY_LOCAL_STORAGE_KEY, ev.target.value);
    }
</script>

{#if alertMessage}
    <div class="alert">{alertMessage}</div>
{/if}

<div class="card">
    <h2>Data</h2>
    <form>
        <div class="form-row">
            <label for="monthSelector">Month</label>
            <select id="monthSelector" bind:value={month}>
                {#each monthNames as monthName, i}
                    <option value={i}>{monthName}</option>
                {/each}
            </select>
        </div>
        <div class="form-row">
            <label for="holidayDaysInput">Holiday</label>
            <input id="holidayDaysInput" type="number" bind:value={holidays} max={daysInMonth}>
        </div>

        <div class="form-row">
            <label for="offDaysInput">Offline days</label>
            <input id="offDaysInput" type="number" min="0" bind:value={offlineDays} max={daysInMonth}>
        </div>

        <div class="form-row">
            <label for="offlineHoursInput">Offline extra hours</label>
            <input id="offlineHoursInput" type="number" min="0" bind:value={offlineHours} max={daysInMonth * 8}>
        </div>

        <div class="form-row">
            <label for="salaryInput">Salary per month</label>
            <input id="salaryInput" type="number" min="0" bind:value={salary} on:change={storeSalary}>
        </div>

        <button on:click|preventDefault={calculate}>Calculate</button>
    </form>
</div>

{#if calculated && !alertMessage}
    <div class="card card-results">
        <h2>Results</h2>
        <div>Date: {currentDate.getFullYear()}, {monthNames[month]}</div>
        <div>Days in the month: {daysInMonth}</div>
        <div>Hours in month: {hoursInMonth}</div>
        <div>Hours worked: {hoursWorked}</div>
        <div>Hours offline: {hoursOff}</div>
        <div class="invoice-value">Invoice: {invoice} $</div>
    </div>
{/if}


<style>
    h2 {
        margin-top: 0;
    }

    .alert {
        padding: 1.5rem;
        border-radius: 5px;
        background-color: tomato;
        color: #fff;
        font-weight: 600;
        text-shadow: #1a1a1a;
        margin-bottom: 1.5rem;
    }
    select, input {
        width: 100%;
        box-sizing: border-box;
    }

    .form-row {
        display: flex;
        flex-direction: column;
        align-content: start;
        align-items: start;
        justify-content: stretch;
        padding-bottom: 1rem;
    }

    .card {
        padding: 1.5rem;
        border-radius: 15px;
        background-color: black;
        margin-bottom: 1.5rem;
        min-width: 30vw;
        }

    .card-results {
        background-color: royalblue;
    }

    .invoice-value {
        padding: .5rem;
        font-weight: bold;
        font-size: 1.5rem;
    }
</style>
