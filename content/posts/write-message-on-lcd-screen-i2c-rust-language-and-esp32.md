+++
title = "Write Messge on LCD Screen With i2C using Rust Language and ESP32-C3"
date = 2024-04-05T22:47:45-03:00
draft = false
+++

Rust is a great language, so since we've heard so far that is one great programming language to build embedded systems, why not create a program
that writes messages on a LCD Screen. In this case we will be using a I2C LCD Screen. A ESP32-C3 and Rust Language using Standard Library and Espressif Framework.

### Rust as a Firmware platform

In comparison to Arduino is way better since you don't depend on limitations of wrappers. In this case you will touch
metal right away.

Let's start by creating a template project; for this you need to install rust. for more information on how to install it
go to https://www.rust-lang.org/

### Let's get our hands dirty

```sh
cargo generate --git https://github.com/esp-rs/esp-idf-template.git --name esp32-c3-lcd
```

enter now on your folder esp32-c3-lcd and look up for the /src/main.rs

now make sure you can build the hello world. by runningthe following command.

```sh
cargo build
```

### Dependencies

after done this and works fine. we now need to modify our dependencies. Cargo.toml

```sh
[dependencies]
log = { version = "0.4", default-features = false }
esp-idf-svc = { version = "0.48", default-features = false }
ssd1306 = "0.8.4"
embedded-graphics = "0.8.1"
```

### Make it work

now Lets update the main.rs with the following code.

```rs
use embedded_graphics::{mono_font::{MonoTextStyle, ascii::FONT_10X20}, pixelcolor::BinaryColor, text::Text, geometry::Point, Drawable, primitives::{Circle, PrimitiveStyle, Primitive}};
use esp_idf_svc::hal::{peripherals::Peripherals, i2c, units::FromValueType, delay::FreeRtos};
use ssd1306::{Ssd1306, size::DisplaySize72x40, rotation::DisplayRotation, I2CDisplayInterface};
use ssd1306::mode::DisplayConfig;

fn main() {

    esp_idf_svc::sys::link_patches();

    let peripherals = Peripherals::take().unwrap();
    let scl = peripherals.pins.gpio6;
    let sda = peripherals.pins.gpio5;

    let config = i2c::config::Config::new().baudrate(FromValueType::kHz(400).into());
    let _i2c = i2c::I2cDriver::new(peripherals.i2c0, sda, scl, &config).unwrap();
    let interface = I2CDisplayInterface::new(_i2c);

    let mut display = Ssd1306::new(interface, DisplaySize72x40, DisplayRotation::Rotate0).into_buffered_graphics_mode();

    display.init().expect("Screen Error");
    let style = MonoTextStyle::new(&FONT_10X20, BinaryColor::On);

    loop {
        // Hello World Text
        println!("Showing hola");
        Text::new("HOLA", Point::new(0, 17), style).draw(&mut display).unwrap();
        display.flush().unwrap();
        display.clear_buffer();
        FreeRtos::delay_ms(2000);

        println!("Showting mundo");
        Text::new("MUNDO", Point::new(0, 17), style).draw(&mut display).unwrap();
        display.flush().unwrap();
        display.clear_buffer();
        FreeRtos::delay_ms(2000);

        // Create Circle
        println!("Showing circle");
        Circle::new(Point::new(36,20), 15).into_styled(PrimitiveStyle::with_stroke(BinaryColor::On, 1)).draw(&mut display).unwrap();
        Circle::new(Point::new(28,18), 3).into_styled(PrimitiveStyle::with_stroke(BinaryColor::On, 1)).draw(&mut display).unwrap();
         Circle::new(Point::new(25, 38), 3).into_styled(PrimitiveStyle::with_stroke(BinaryColor::On, 1)).draw(&mut display).unwrap();

        display.flush().unwrap();
        display.clear_buffer();
        FreeRtos::delay_ms(2000);

    }
}
```

now as you see we created A text and a Circle. take into account that you need to configure your baudrate, as well your
scl and sda Gpio pin peripherals.

### Execute!

Now we need to run the following commands.

```sh
cargo build && cargo flash && cargo flash monitor
```
