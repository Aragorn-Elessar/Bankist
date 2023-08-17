<a name="readme-top"></a>

# Bankist App Project

## Table of Contents

- [Project-Description](#Project-Description)
- [Prerequisites](#Prerequisites)
- [Installing](#Installing)
- [Steps](#Steps)
- [Author](#Author)
- [Credits](#Credits)

## Project-Description

The starter project had the page structure and interface set in HTML and CSS files for the Bankist page along with the accounts data in the JS file. I needed to provide a fully functioning app for the account owners by handling their data to implement a user log in with full detailed interactive interface that includes some tools like money transfer/loan and close account requests.

## Prerequisites

Any code editor (e.g: VSCode, Atom,... etc)

## Installing

Terminal commands to start using project:

- Get a copy on your machine

```
`git clone https://github.com/Aragorn-Elessar/Bankist.git`
```

- Call into the directory location

```
`cd Bankist`
```

- Opens code in `VSCode`

```
`code .`
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Steps

- Define selectors for needed elements (labels/buttons/inputs)

```js
const labelWelcome = document.querySelector('.welcome');
const labelDate = document.querySelector('.date');
const labelBalance = document.querySelector('.balance__value');
const labelSumIn = document.querySelector('.summary__value--in');
const labelSumOut = document.querySelector('.summary__value--out');
const labelSumInterest = document.querySelector('.summary__value--interest');
const labelTimer = document.querySelector('.timer');

const containerApp = document.querySelector('.app');
const containerMovements = document.querySelector('.movements');

const btnLogin = document.querySelector('.login__btn');
const btnTransfer = document.querySelector('.form__btn--transfer');
const btnLoan = document.querySelector('.form__btn--loan');
const btnClose = document.querySelector('.form__btn--close');
const btnSort = document.querySelector('.btn--sort');

const inputLoginUsername = document.querySelector('.login__input--user');
const inputLoginPin = document.querySelector('.login__input--pin');
const inputTransferTo = document.querySelector('.form__input--to');
const inputTransferAmount = document.querySelector('.form__input--amount');
const inputLoanAmount = document.querySelector('.form__input--loan-amount');
const inputCloseUsername = document.querySelector('.form__input--user');
const inputClosePin = document.querySelector('.form__input--pin');
```

### Helper Functions

- Format date for account displayed movements

```js
// Calculate date
const formatMovementDate = function (date, locale) {
  const calcDaysPassed = (date1, date2) =>
    Math.round(Math.abs(date2 - date1) / (1000 * 60 * 60 * 24));

  const daysPassed = calcDaysPassed(new Date(), date);

  if (daysPassed === 0) return 'Today';
  else if (daysPassed === 1) return 'Yesterday';
  else if (daysPassed <= 7) return `${daysPassed} days ago`;

  return new Intl.DateTimeFormat(locale).format(date);
};
```

- Format numbers using internationalizing API for money displaying

```js
// Display numbers using internationalized format
const formatNumber = function (num, currency, locale) {
  return new Intl.NumberFormat(locale, {
    style: 'currency',
    currency: currency,
  }).format(num);
};
```

- Display account deposite/withdrawal movements including any money tranfer/loan transaction along with `sort` functionality

```js
// Display deposite/withdrawal movements for account
const displayMovements = function (acc, sort = false) {
  containerMovements.innerHTML = '';

  const movs = sort
    ? acc.movements.slice().sort((a, b) => a - b)
    : acc.movements;

  movs.forEach((mov, i) => {
    const type = mov > 0 ? 'deposit' : 'withdrawal';

    const date = new Date(acc.movementsDates[i]);
    const displayDate = formatMovementDate(date, acc.locale);

    const displayNumber = formatNumber(mov, acc.currency, acc.locale);

    const html = `
    <div class="movements__row">
      <div class="movements__type movements__type--${type}">${
      i + 1
    } ${type}</div>
    <div class="movements__date">${displayDate}</div>
      <div class="movements__value">${displayNumber}</div>
    </div>
    `;

    containerMovements.insertAdjacentHTML('afterbegin', html);
  });
};
```

- Calculate and display total balance using `formatNumber()` function

```js
// Calculate and display total balance
const calcDisplayBalance = function (acc) {
  acc.balance = acc.movements.reduce((acc, mov) => acc + mov, 0);
  labelBalance.textContent = `${formatNumber(
    acc.balance,
    acc.currency,
    acc.locale
  )}`;
};
```

- Calculate and display summary for in/out/interest

```js
// Display summary for in/out/interest
const calcDisplaySummary = function (acc) {
  const incomes = acc.movements
    .filter(mov => mov > 0)
    .reduce((acc, mov) => acc + mov);
  labelSumIn.textContent = `${formatNumber(
    acc.balance,
    acc.currency,
    acc.locale
  )}`;

  const outcomes = acc.movements
    .filter(mov => mov < 0)
    .reduce((acc, mov) => acc + mov);
  labelSumOut.textContent = `${formatNumber(
    Math.abs(outcomes),
    acc.currency,
    acc.locale
  )}`;

  // Bank gives a percentage interest for each deposit if the interest is more than 1€
  const interest = acc.movements
    .filter(mov => mov > 0)
    .map(deposite => (deposite * acc.interestRate) / 100)
    .filter(int => int >= 1)
    .reduce((acc, int) => acc + int);
  labelSumInterest.textContent = `${formatNumber(
    interest,
    acc.currency,
    acc.locale
  )}`;
};
```

- Create usernames for account owners using the accounts array

```js
// Create usernames for account owners
const createUsernames = function (accs) {
  accs.forEach(
    acc =>
      (acc.username = acc.owner
        .toLowerCase()
        .split(' ')
        .map(name => name[0])
        .join(''))
  );
};
createUsernames(accounts);
```

- Update user interface including movements/balance/summary

```js
const updateUI = function (acc) {
  // Display movements
  displayMovements(acc);

  // Display balance
  calcDisplayBalance(acc);

  // Display summary
  calcDisplaySummary(acc);
};
```

- Log out timer to log the current user out when they idles for 2 min

```js
// Logout count down timer function
const startLogOutTimer = function () {
  const tick = () => {
    const min = String(Math.trunc(time / 60)).padStart(2, 0);
    const sec = String(time % 60).padStart(2, 0);

    // In each call, print the remaining time to UI
    labelTimer.textContent = `${min}:${sec}`;

    // When 0 seconds, stop timer and log out user
    if (time === 0 || owner !== currentAccount) {
      clearInterval(timer);
      labelWelcome.textContent = 'Log in toget started';
      containerApp.style.opacity = 0;
    }

    // Decrease 1s
    time--;
  };

  // Set time to 2 min
  let time = 120;
  let owner = currentAccount;

  // Call the timer immediately to avoid the delay
  tick();
  // Call the timer every second
  const timer = setInterval(tick, 1000);

  // To use the clearInterval function, we need the timer variable
  return timer;
};
```

### Event Handlers

- Global variables to maintain current logged user and timer state

```js
let currentAccount, timer;
```

- User login functionality

```js
btnLogin.addEventListener('click', function (e) {
  // Prevent form from submitting
  e.preventDefault();

  currentAccount = accounts.find(
    acc => acc.username === inputLoginUsername.value
  );

  currentAccount?.pin === Number(inputLoginPin.value);

  // Clear input fields
  inputLoginPin.value = inputLoginUsername.value = '';
  inputLoginPin.blur();

  // Display UI and message
  labelWelcome.textContent = `Welcome back, ${
    currentAccount.owner.split(' ')[0]
  }`;
  containerApp.style.opacity = 100;

  // Create current date and time
  const now = new Date();
  const options = {
    hour: 'numeric',
    minute: 'numeric',
    day: 'numeric',
    month: 'numeric',
    year: 'numeric',
  };
  // const locale = navigator.language;

  labelDate.textContent = new Intl.DateTimeFormat(
    currentAccount.locale,
    options
  ).format(now);

  // Timer
  if (timer) clearInterval(timer);
  timer = startLogOutTimer();

  // Update UI
  updateUI(currentAccount);
});
```

- Money transfer functionality from current to targeted account

```js
// Money transfer functionality
btnTransfer.addEventListener('click', function updateBalance(e) {
  // Prevent from submitting
  e.preventDefault();
  const moneyRecipient = accounts.find(
    acc => acc.username === inputTransferTo.value
  );
  const amount = Number(inputTransferAmount.value);

  // Clear input fields
  inputTransferTo.value = inputTransferAmount.value = '';
  inputTransferAmount.blur();

  if (
    amount > 0 &&
    moneyRecipient &&
    currentAccount.balance >= amount &&
    moneyRecipient.username !== currentAccount.username
  ) {
    // Update sender/recipient movements
    currentAccount.movements.push(-amount);
    moneyRecipient.movements.push(amount);

    // Add transfer date
    currentAccount.movementsDates.push(new Date().toISOString());
    moneyRecipient.movementsDates.push(new Date().toISOString());

    // Update UI
    updateUI(currentAccount);

    // Reset timer
    clearInterval(timer);
    timer = startLogOutTimer();
  }
});
```

- Loan request handler including a `setTimeout()` timer to simulate a needed time to check the availability with the bank, reset logout timer, and update UI

```js
// Request loan
btnLoan.addEventListener('click', function provideLoan(e) {
  e.preventDefault();

  const loan = Math.floor(inputLoanAmount.value);

  // Grant loan if one deposit with at least 10% of the requested loan amount exists
  if (loan > 0 && currentAccount.movements.some(mov => mov >= loan * 0.1)) {
    // Delay 2.5 sec before granting loan
    setTimeout(() => {
      // Add movement
      currentAccount.movements.push(loan);

      // Add loan date
      currentAccount.movementsDates.push(new Date().toISOString());

      // Update UI
      updateUI(currentAccount);
    }, 2500);

    // Reset timer
    clearInterval(timer);
    timer = startLogOutTimer();
  }

  // Clear input fields
  inputLoanAmount.value = inputClosePin.value = '';
  inputLoanAmount.blur();
});
```

- Close account handler if the logged user provided the correct account data

```js
// Close account handling
btnClose.addEventListener('click', function deleteAccount(e) {
  e.preventDefault();

  // Delete user account
  if (
    inputCloseUsername.value === currentAccount.username &&
    Number(inputClosePin.value) === currentAccount.pin
  ) {
    const index = accounts.findIndex(
      acc => acc.username === currentAccount.username
    );
    accounts.splice(index, 1);

    // Hide UI
    containerApp.style.opacity = 0;
  }

  // Clear input fields
  inputCloseUsername.value = inputClosePin.value = '';
  inputClosePin.blur();
});
```

- Close account handler if the logged user provided the correct account username & password

```js
// Close account handling
btnClose.addEventListener('click', function deleteAccount(e) {
  e.preventDefault();

  // Delete user account
  if (
    inputCloseUsername.value === currentAccount.username &&
    Number(inputClosePin.value) === currentAccount.pin
  ) {
    const index = accounts.findIndex(
      acc => acc.username === currentAccount.username
    );
    accounts.splice(index, 1);

    // Hide UI
    containerApp.style.opacity = 0;
  }

  // Clear input fields
  inputCloseUsername.value = inputClosePin.value = '';
  inputClosePin.blur();
});
```

- Sort movments in a descending order on sort button click

```js
// Sort movements in a descending order
// reserve sort state to be false by default
let sorted = false;
btnSort.addEventListener('click', function (e) {
  e.preventDefault();
  displayMovements(currentAccount, sorted ? (sorted = false) : (sorted = true));
});
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Author

[Mahmoud Gadallah](https://github.com/Aragorn-Elessar)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

## Credits

A [Udemy](https://www.udemy.com) project, provided by [Jonas Schmedtmann](https://www.udemy.com/user/jonasschmedtmann/) JavaScript course

<p align="right">(<a href="#readme-top">back to top</a>)</p>
