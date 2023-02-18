# Detailed Clock Reference
This section seeks to cover information about the clocking options on the tinyAVR parts at somewhat greater depth than a simple list of supported options.

## How the Internal clock works, briefly
The OSCCFG fuse selects whether the speed is derived from a 20 MHz or 16 MHz oscillator. At power-on, these are always set to a prescaler of /6 hence 2.66 or 3.33 MHz. The core, prior to calling setup, will reconfigure the prescaler to match the requested speed. The OSCCFG fuse is always set when a sketch is uploaded via a UPDI programmer.
They additionally have the equivalent of the classic AVR's OSCCAL calibration register, and like those parts, the compliance is many times what the datasheet claims, and you can get a very wide range of speeds from them. 0/1-series has 64 settings for each nominal clock frequency, and 2-series parts have 128. Determining the value to set it to in order to obtain a desired clock speed is called "Tuning", even though you're not doing it for accuracy (it's generally already at the best setting from factory cal if you want the nominal clock speed.)

## Supported Clock Speeds
Like classic AVRs, these have "speed grades" depending on the voltage and operating conditions. The spec is 5 MHz @ 1.8V , 10 MHz @ 2.7V (3.3V nominal) and 20 @ 4.5V (5.0V nominal). See the Speed Grade reference for more information on this. The speed grades are extremely conservative.



| Clock Speed | Within Spec | Internal | Ext.Clk |Tuned| 0/1 |Tuned|  2  | Notes |
|-------------|-------------|----------|---------|-----|-----|-----|-----|-------|
|       1 MHz |         Yes |      Yes |      No | Yes |Presc| Yes |Presc|       |
|       2 MHz |         Yes |      Yes |      No | Yes |Presc| Yes |Presc|       |
|    2.66 MHz |         Yes |       No |      No |  No | N/A | N/A | N/A | 2     |
|       3 MHz |         Yes |       No |      No | Yes |Presc| Yes |Presc| 3     |
|    3.33 MHz |         Yes |       No |      No |  No | N/A | N/A | N/A | 2     |
|       4 MHz |         Yes |      Yes |      No | Yes |Presc| Yes |Presc|       |
|       6 MHz |         Yes |       No |      No | Yes |Presc| Yes |Presc| 3     |
|       7 MHz |         Yes |       No |      No | Yes |Presc| Yes*|Presc| 3,4,9 |
|       8 MHz |         Yes |      Yes |     Yes | Yes |Presc| Yes |Presc|       |
|      10 MHz |         Yes |      Yes |     Yes | Yes |Presc| Yes |Presc| 5     |
|      12 MHz |         Yes |       No |     Yes | Yes | Yes | Yes |Yes* | 5     |
|      14 MHz |         Yes |       No |      No | Yes | Yes | Yes|  No | 4,9    |
|      16 MHz |         Yes |      Yes |     Yes | ofc | Yes | ofc | Yes | 6     |
|      20 MHz |         Yes |      Yes |     Yes | Yes | ofc | Yes | ofc | 6     |
|      24 MHz |          No |       No |     Yes | Yes | Yes | Yes | Yes | 7     |
|      25 MHz |          No |       No |     Yes | Yes | Yes | Yes | Yes | 7     |
|      28 MHz |          No |       No |     Yes |  No |  No |  No |  No | 4     |
|      30 MHz |          No |       No |     Yes |Maybe| Yes |Maybe| Yes | 5, 8  |
|      32 MHz |          No |       No |     Yes |  No | Yes |Maybe| Yes | 5, 8  |

Notes:
1. 2 MHz could be made by prescaling, ~but there is no demand~ and was added in 2.6.0
2. Optiboot always runs at 2.66 or 3.33 MHz (it does not unset the /6 prescaler). Depending on OSCCFG, it always starts up at one of these speeds
3. This speed could be generated by prescaling a tuned oscillator, but we don't support this because there is no convincing need.
4. This speed, or the speed that a tuned oscillator would need to run at to be prescaled to it, is an awkward unpopular frequency for which there is no demand.
5. Where marked with an asterisk, this speed is at the edge of what is possible with a tuned oscillator - not all chips' oscillators will get this high. In the case of the speeds at the low end, we make up for it by prescaling double that frequency if we have to; obviously that doesn't work for the high end of the range.
6. 20 MHz is well within tuning range of the 16 MHz oscillators and vice versa. This is particularly useful if using Optiboot and hence not setting clock on upload.
7. This is a mild overclock, and it will almost always work at room temperature and 5v.
8. This is an aggressive overclock. Best results can be had by using F-spec (extended temperature range) parts; solid 5v supply is a must. Higher speeds can generally be achieved with an external clock.
9. 14 and 7 MHz, for people who like making life harder for themselves, can be achieved via tuning, but for the 2-series, `OSCCFG` must be set for 16 MHz

megaTinyCore has no plans to add support for clock speeds that do not approximate an integer (ex, 1.25 MHz, 2.5 MHz - and certainly not, the technically achievable (20/48), but useless, 516 kHz. )

The tinyAVR product line does not support use of an external high frequency crystal, only external clock or the internal oscillator can be used. The internal oscillator is pretty accurate, so internal clock will work fine for UART communication()they're within a fraction of a percent at room temp) with very little voltage dependence.

## Tuning
The compliance of the tinyAVR 0/1/2-series oscillator blows the datasheet specifications out of the water and into low earth orbit. The difference between the speed with the oscillator frequency cal at it's minimum and at it's maximum values is about the magnitude of it's nominal frequency! 0/1-series parts usually go from a little under 5/8ths of their nominal frequency with tuning at 0 up to 1-5/8ths with tuning at the maximum! 2-series parts have a slightly wider range, centered a couple of MHz higher; a typical 2-series, at a nominal frequency of 20 MHz could be tuned from as low as 12 MHz up to around 36 or so, if you were to extrapolate the speed speed trend (they generally don't make it all the way to 0x7F tuning).

A tuning sketch must be used to experimentally determine the values to use. See the [Tuning Reference](https://github.com/SpenceKonde/megaTinyCore/blob/master/megaavr/extras/Ref_Tuning.md). Tuning is a LOT more fun here than on the DX-series!

## External clocks
External clocks are always an option. The layout considerations are pretty simple: Keep the high frequency trace as short as possible. Don't connect anything else to the high frequency trace, make sure you follow the guidelines in their datasheet on decoupling capacitors. And if using one of the rectangular SMT ones, be careful to orient it correctly when placing the part. It fits just fine rotated 180 degrees, which will connect vcc and gnd backwards, instantly destroying the oscillator and presenting a near short to the power supply.

There are a lot of reasons, however, to not use them:
1. They are power hogs. 10-25 mA is normal, 50mA is not unheard-of.
2. They are picky about voltage. There was a line of oscillators in 5032 package in production until mid 2020 which worked at 1.8-5v. They were discontinued, and there is no other such oscillator available.
  a. The 5v ones specify minimum 4.5v maximum 5.5v.
  b. 1.6-3.6v ones are available, so the lower end of the range is covered. None of the major vendors with parametric search on their catalog indicate that any which are in spec operating between 3.6v and 4.5V, so one cannot run directly from a LiPo battery with one while remaining within spec.
3. 5v units are not available in packages smaller than 7050 (7mm x 5mm) typically, 5032 is exotic for 5v oscillators, and anything smaller nonexistent.
4. They are strangely expensive, typically $1 or more from western suppliers. There is hardly any "China discount", either. Like crystals, however, the oscillators advertised on Aliexpress don't list essential specs like the voltage or part number, and sellers don't seem to know when asked.
  a. One gets the impression that external oscillators are a specialty item for precision applications, while crystals are not. They are often designed to higher accuracy and they don't depend on external components that could "pull" the frequency, and so on. In a typical application, if you don't need a precision clock source, there are other ways to get it.


## Clock troubleshooting
Thankfully the tinyAVR clock system has few things that can go wrong. As noted above, when uploading via UPDI, the clock fuse is set automatically. Thus, when uploading via UPDI, and using the internal oscillator without tuning, it's pretty much foolproof. When using Optiboot, you do need to take care to select the clock speed that matches what you selected when you burned the bootloader (or run the tuning sketch, and use the "Tuned" settings).

External Clocks are also fairly hard to mess up, too - they're nothing like crystals, which require you to calculate the value of loading capacitors based on numbers you don't know and don't have the equipment to measure, or more likely, try to guess pick a value that will be good enough, and be ready to swap out the loading caps in event that it's not.

### Failure modes

#### Tuned Internal Oscillator
There are two general failure modes here. First, you may discover that it is operating, but not at the expected speed. Did you run the tuning sketch, or are you relying on the core to guess? The guessing is not 100% effective. If the speed is WAY off, as in, something like 2.66 MHz or 3.33 MHz, it has either been tuned but found not to reach the requested speed. This usually happens when trying to run parts at the highest supported tuned overclock, and not all oscillators are equal. Some skew low, others skew high. The ones that are lower than average can't hit the top speeds. Alternately, it may have been tuned, and in the process the tuning sketch detected that the chip was not stable at that speed and marked it as unreachable. In either of those cases, use a slower speed, or a different chip; the one you're using can't reach the speed you want.

If, instead, the speed is close to what you asked for, but off a few percent, I would suspect that the chip was not tuned properly and repeat the tuning process.

The other failure mode is that it does seem to run at the requested speed, but you notice that sketches hang over time, or occasionally output nonsensical numbers (typically 1's turn into 0's). This means that that chip is being run faster than it's capable of under the voltage and temperature conditions. Do you have a stable 5V supply that is well-decoupled? That is essential.
You may just have to use a different chip, as in the case above.

Considerations for Tuned oscillators are covered in greater depth in the [Tuning Reference](https://github.com/SpenceKonde/megaTinyCore/blob/master/megaavr/extras/Ref_Tuning.md).

### External clocks
The normal failure mode observed is that the chip just *stops* very rarely in the initialization process - it switches to the external clock, ceases to have a clock, and doesn't do anything more. There aren't many possible causes.
Unlike crystals, you can actually put a 'scope probe onto the output of a the oscillator and see the signal. Do that. Do you see the signal?
1. If not - examine the datasheet and try to figure out why it might not be oscillating.
  a. Verify that it is getting power and ground connected to the proper pins. (on my breakout board which support either a crtystal or external clock, you need to complete a solder bridge to  connect it to Vcc, so you can also use it with a crystal)/
  b. Most have a pin that can be pulled low to turn off the output to save power. If allowed to float, they generally have internal pullups and can be left alone, but putting a 10k pullup to Vcc is an easy test to verify. Is yours shorted to ground by any chance?
  c. You didn't install it rotated 180 degrees did you? the rectangular oscillators usually have Vcc and ground opposite each other, so everything will be lined up and can be soldered down with the oscillator connected backwards. This will irrevocably destroy it within seconds. You can easily determine if this has happened - the part will be hot to the touch,
2. If you do see a signal, and it is no higher than 24 MHz (ie, there's no chance it's actually too fast for the chip), make sure you see the same signal on the pin of the chip (cold solder joints happen - I've seen parts with a pin that wasn't actually soldered in place, but which looked like it was, and when pressure was applied to the pin with the scope probe - it started working; this is much more frustrating when you're trying to debug it with the power off with a DMM, and you check every connection, yup, continuity.... but when you remove the probe and connect power, it no longer makes contact )