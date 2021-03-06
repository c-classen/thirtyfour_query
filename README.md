[![Crates.io](https://img.shields.io/crates/v/thirtyfour_query.svg?style=for-the-badge)](https://crates.io/crates/thirtyfour_query)
[![docs.rs](https://img.shields.io/badge/docs.rs-thirtyfour_query-blue?style=for-the-badge)](https://docs.rs/thirtyfour_query)

Advanced element query/polling interfaces for the thirtyfour crate.

These interfaces are currently experimental and will likely be merged into
the main `thirtyfour` crate once they are stable. 

If you can help by testing these and providing feedback that would be appreciated.

## Usage

### ElementQuery 

First, import the following:
```rust
use thirtyfour_query::{ElementPoller, ElementQueryable};
```

Next, set the default polling behaviour:
```rust 
// Disable implicit timeout in order to use new query interface.
driver.set_implicit_wait_timeout(Duration::new(0, 0)).await?;

let poller = ElementPoller::TimeoutWithInterval(Duration::new(20, 0), Duration::from_millis(500));
driver.config_mut().set("ElementPoller", poller)?;
```

Other ElementPoller options are also available, such as NoWait and NumTriesWithInterval.
These can be overridden on a per-query basis as needed.

Now, using the query interface you can do things like:

```rust
let elem_text = 
    driver.query(By::Css("thiswont.match")).or(By::Id("searchInput")).first().await?;
```
    
This will execute both queries once per poll iteration and return the first one that matches.
You can also filter on one or both match arms like this:

```rust
driver.query(By::Css("thiswont.match")).with_text("testing")
    .or(By::Id("searchInput")).with_class("search").and_not_enabled()
    .first().await?;
```

To fetch all matching elements instead of just the first one, simply change first() to all() 
and you'll get a Vec instead.

ElementQuery also allows the user of custom predicates that take a `&WebElement` argument
and return a `WebDriverResult<bool>`.

### ElementWaiter

First, import the following:
```rust
use thirtyfour_query::{ElementPoller, ElementWaitable};
```

Next, set the default polling behaviour (same as for ElementQuery - the same polling
settings are used for both):
```rust 
// Disable implicit timeout in order to use new query interface.
driver.set_implicit_wait_timeout(Duration::new(0, 0)).await?;

let poller = ElementPoller::TimeoutWithInterval(Duration::new(20, 0), Duration::from_millis(500));
driver.config_mut().set("ElementPoller", poller)?;
```

Now you can do things like this:
```rust
elem.wait("Timed out waiting for element to be displayed").until().displayed().await?;
elem.wait("Timed out waiting for element to disappear").until_not().displayed().await?;

elem.wait("Timed out waiting for element to become enabled").until().enabled().await?;
elem.wait("Timed out waiting for element to become disabled").until_not().enabled().await?;
```

And so on, including `selected()` and `stale()`.

ElementWaiter also allows the user of custom predicates that take a `&WebElement` argument
and return a `WebDriverResult<bool>`.

## LICENSE

This work is dual-licensed under MIT or Apache 2.0.
You can choose either license if you use this work.

`SPDX-License-Identifier: MIT OR Apache-2.0`
