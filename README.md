## PHP Shipping API

![Test Coverage](http://img.shields.io/badge/coverage-86%25-green.svg)
[![Build Status](https://travis-ci.org/pdt256/shipping.svg?branch=master)](https://travis-ci.org/pdt256/shipping)
[![Downloads](https://img.shields.io/packagist/dt/pdt256/shipping.svg)](https://packagist.org/packages/pdt256/shipping)
[![License](https://img.shields.io/packagist/l/pdt256/shipping.svg)](https://github.com/pdt256/shipping/blob/master/LICENSE.txt)

A shipping rate wrapper for USPS, UPS, and Fedex.

## Introduction

This is a PHP shipping package that wraps API calls to UPS, FedEx, and USPS for shipping rates.
Multiple packages can be added to get additional rates.

All code (including tests) conform to the PSR-2 coding standards.
The namespace and autoloader are using the PSR-4 standard.

All pull requests are processed by Travis CI to conform to PSR-2 and to verify all unit tests pass. 

## Installation

Add the following lines to your ``composer.json`` file.

```JSON
{
    "require": {
        "pdt256/shipping": "1.0.*@dev"
    }
}
```

## Example

Create a shipment object:

```php
$shipment = new Shipment;
$shipment
    ->setFromIsResidential(false)
    ->setFromStateProvinceCode('IN')
    ->setFromPostalCode('46205')
    ->setFromCountryCode('US')
    ->setToIsResidential(true)
    ->setToPostalCode('20101')
    ->setToCountryCode('US');

$package = new Package;
$package
    ->setLength(12)
    ->setWidth(4)
    ->setHeight(3)
    ->setWeight(3);

$shipment->addPackage($package);
```

## UPS (Stub) Example

Below is an example request to get shipping rates from the UPS API. 

Notice: The below line uses a stub class to fake a response from the UPS API.
You can immediately use this method in your code until you get an account with UPS.

```php
'requestAdapter' => new RateRequest\StubUPS(),
```

```php
use pdt256\Shipping\UPS;
use pdt256\Shipping\RateRequest;

$ups = new UPS\Rate([
    'prod'           => false,
    'accessKey'      => 'XXXX',
    'userId'         => 'XXXX',
    'password'       => 'XXXX',
    'shipperNumber'  => 'XXXX',
    'shipment'       => $shipment,
    'approvedCodes'  => [
        '03', // 1-5 business days
        '02', // 2 business days
        '01', // next business day 10:30am
        '13', // next business day by 3pm
        '14', // next business day by 8am
    ],
    'requestAdapter' => new RateRequest\StubUPS(),
]);

$rates = $ups->getRates();
```

Output array sorted by cost: (in cents)

```php
array (
  0 => 
  pdt256\Shipping\Quote::__set_state(array(
     'code' => '03',
     'name' => 'UPS Ground',
     'cost' => 1910,
     'transitTime' => NULL,
     'deliveryEstimate' => NULL,
     'carrier' => 'ups',
  )),
  1 => 
  pdt256\Shipping\Quote::__set_state(array(
     'code' => '02',
     'name' => 'UPS 2nd Day Air',
     'cost' => 4923,
     'transitTime' => NULL,
     'deliveryEstimate' => NULL,
     'carrier' => 'ups',
  )),
  2 => 
  pdt256\Shipping\Quote::__set_state(array(
     'code' => '13',
     'name' => 'UPS Next Day Air Saver',
     'cost' => 8954,
     'transitTime' => NULL,
     'deliveryEstimate' => NULL,
     'carrier' => 'ups',
  )),
  3 => 
  pdt256\Shipping\Quote::__set_state(array(
     'code' => '01',
     'name' => 'UPS Next Day Air',
     'cost' => 9328,
     'transitTime' => NULL,
     'deliveryEstimate' => NULL,
     'carrier' => 'ups',
  )),
)
```

## USPS (Stub) Example

```php
use pdt256\Shipping\USPS;
use pdt256\Shipping\RateRequest;

$usps = new USPS\Rate([
	'prod'     => false,
	'username' => 'XXXX',
	'password' => 'XXXX',
	'shipment' => $shipment,
	'approvedCodes'  => [
		'1', // 1-3 business days
		'4', // 2-8 business days
	],
	'requestAdapter' => new RateRequest\StubUSPS(),
]);

$rates = $usps->getRates();
```

Output array sorted by cost: (in cents)

```php
array (
  0 => 
  pdt256\Shipping\Quote::__set_state(array(
     'code' => '4',
     'name' => 'Parcel Post',
     'cost' => 1001,
     'transitTime' => NULL,
     'deliveryEstimate' => NULL,
     'carrier' => 'usps',
  )),
  1 => 
  pdt256\Shipping\Quote::__set_state(array(
     'code' => '1',
     'name' => 'Priority Mail',
     'cost' => 1220,
     'transitTime' => NULL,
     'deliveryEstimate' => NULL,
     'carrier' => 'usps',
  )),
)
```

## Fedex (Stub) Example

```php
use pdt256\Shipping\Fedex;
use pdt256\Shipping\RateRequest;

$fedex = new Fedex\Rate([
	'prod'           => FALSE,
	'key'            => 'XXXX',
	'password'       => 'XXXX',
	'accountNumber' => 'XXXX',
	'meterNumber'   => 'XXXX',
	'dropOffType'  => 'BUSINESS_SERVICE_CENTER',
	'shipment'       => $shipment,
	'approvedCodes'  => [
		'FEDEX_EXPRESS_SAVER',  // 1-3 business days
		'FEDEX_GROUND',         // 1-5 business days
		'GROUND_HOME_DELIVERY', // 1-5 business days
		'FEDEX_2_DAY',          // 2 business days
		'STANDARD_OVERNIGHT',   // overnight
	],
	'requestAdapter' => new RateRequest\StubFedex(),
]);

$rates = $fedex->getRates();
```

Output array sorted by cost: (in cents)

```php
array (
  0 => 
  pdt256\Shipping\Quote::__set_state(array(
     'code' => 'GROUND_HOME_DELIVERY',
     'name' => 'Ground Home Delivery',
     'cost' => 1655,
     'transitTime' => 'THREE_DAYS',
     'deliveryEstimate' => NULL,
     'carrier' => 'fedex',
  )),
  1 => 
  pdt256\Shipping\Quote::__set_state(array(
     'code' => 'FEDEX_EXPRESS_SAVER',
     'name' => 'Fedex Express Saver',
     'cost' => 2989,
     'transitTime' => NULL,
     'deliveryEstimate' => 
    DateTime::__set_state(array(
       'date' => '2014-09-30 20:00:00',
       'timezone_type' => 3,
       'timezone' => 'UTC',
    )),
     'carrier' => 'fedex',
  )),
  2 => 
  pdt256\Shipping\Quote::__set_state(array(
     'code' => 'FEDEX_2_DAY',
     'name' => 'Fedex 2 Day',
     'cost' => 4072,
     'transitTime' => NULL,
     'deliveryEstimate' => 
    DateTime::__set_state(array(
       'date' => '2014-09-29 20:00:00',
       'timezone_type' => 3,
       'timezone' => 'UTC',
    )),
     'carrier' => 'fedex',
  )),
  3 => 
  pdt256\Shipping\Quote::__set_state(array(
     'code' => 'STANDARD_OVERNIGHT',
     'name' => 'Standard Overnight',
     'cost' => 7834,
     'transitTime' => NULL,
     'deliveryEstimate' => 
    DateTime::__set_state(array(
       'date' => '2014-09-26 20:00:00',
       'timezone_type' => 3,
       'timezone' => 'UTC',
    )),
     'carrier' => 'fedex',
  )),
)
```

## Unit Tests:

```bash
    vendor/bin/phpunit
```

### With Code Coverage:

```bash
    vendor/bin/phpunit --coverage-text --coverage-html coverage_report
```

### With Live API Tests:

```bash
    ./live_phpunit.sh
```

## Run Coding Standards Test:

```bash
    vendor/bin/phpcs --standard=PSR2 src/ tests/
```


### License

The MIT License (MIT)

Copyright (c) 2014 Jamie Isaacs <pdt256@gmail.com>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
