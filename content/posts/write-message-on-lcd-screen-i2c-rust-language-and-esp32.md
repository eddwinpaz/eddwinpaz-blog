+++
title = "Write Messge on LCD Screen With i2C using Rust Language and ESP32-C3"
date = 2024-04-05T22:47:45-03:00
draft = false
+++

Rust stands out as an excellent language, particularly for developing embedded systems. Considering its prowess in this domain, why not embark on a project to create a program that displays messages on an LCD screen? Specifically, we'll utilize an I2C LCD Screen, along with an ESP32-C3 microcontroller, leveraging Rust's Standard Library and the Espressif Framework.

Leveraging Rust for Firmware Development
In contrast to Arduino, Rust offers distinct advantages by eliminating the need for cumbersome wrappers. With Rust, you dive straight into interacting with hardware at a low level.

Let's kickstart this endeavor by setting up a template project. To begin, ensure Rust is installed. For detailed installation instructions, visit Rust's official website." go to https://www.rust-lang.org/

### Let's get our hands dirty

```sh
cargo generate --git https://github.com/esp-rs/esp-idf-template.git --name esp32-c3-lcd
```

Navigate to the 'esp32-c3-lcd' folder and locate the '/src/main.rs' file.

Now, confirm that you can successfully build the 'hello world' program by running the following command:

```sh
cargo build
```

### Dependencies

After confirming the successful build of the 'hello world' program, it's time to update our dependencies in the 'Cargo.toml' file

```sh
[dependencies]
log = { version = "0.4", default-features = false }
esp-idf-svc = { version = "0.48", default-features = false }
ssd1306 = "0.8.4"
embedded-graphics = "0.8.1"
```

### Make it work

Now, let's update the 'main.rs' file with the following code

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

Upon inspection, you'll notice that we've created a text and a circle. It's important to configure your baud rate, as well as your SCL and SDA GPIO pin peripherals to ensure proper functionality.

### Execute!

Now, execute the following commands:

```sh
cargo build && cargo flash && cargo flash monitor
```
