# Swaptions-Collars
Valued interest rate derivatives using QuantLib in Python, including European swaptions under the Black model by building a flat term structure, constructing vanilla interest rate swaps, and modeling implied volatility surfaces.

import QuantLib as ql

# Setup
calendar = ql.TARGET()
settlement_date = ql.Date(10, 6, 2025)
ql.Settings.instance().evaluationDate = settlement_date

# Flat Yield Curve (2%)
flat_rate = ql.FlatForward(settlement_date, 0.02, ql.Actual365Fixed())
yield_curve = ql.YieldTermStructureHandle(flat_rate)

# Swaption parameters
exercise_date = settlement_date + ql.Period("1Y")  # European expiry in 1Y
exercise = ql.EuropeanExercise(exercise_date)
maturity = ql.Period("5Y")
notional = 1_000_000

# Schedule
start = exercise_date
end = calendar.advance(start, maturity)
schedule = ql.Schedule(start, end, ql.Period("1Y"), calendar,
                       ql.ModifiedFollowing, ql.ModifiedFollowing,
                       ql.DateGeneration.Forward, False)

index = ql.Euribor6M(yield_curve)

# Receiver Swaption (strike = 2.0%)
receiver_swap = ql.VanillaSwap(ql.VanillaSwap.Receiver,
                               notional, schedule, 0.02, ql.Actual365Fixed(),
                               schedule, index, 0.0, ql.Actual365Fixed())
receiver_swaption = ql.Swaption(receiver_swap, exercise)

# Payer Swaption (strike = 3.0%)
payer_swap = ql.VanillaSwap(ql.VanillaSwap.Payer,
                            notional, schedule, 0.02, ql.Actual365Fixed(),
                            schedule, index, 0.0, ql.Actual365Fixed())
payer_swaption = ql.Swaption(payer_swap, exercise)

# Volatility
vol = 0.20
black_vol = ql.ConstantSwaptionVolatility(settlement_date, calendar,
                                          ql.ModifiedFollowing,
                                          vol,
                                          ql.Actual365Fixed())
vol_handle = ql.SwaptionVolatilityStructureHandle(black_vol)

# Engine
engine = ql.BlackSwaptionEngine(yield_curve, vol_handle)

receiver_swaption.setPricingEngine(engine)
payer_swaption.setPricingEngine(engine)

# Results
receiver_price = receiver_swaption.NPV()
payer_price = payer_swaption.NPV()
collar_price = receiver_price - payer_price

print(f"Receiver Swaption (strike 2.0%): {receiver_price:,.2f}")
print(f"Payer Swaption (strike 3.0%): {payer_price:,.2f}")
print(f"Swaption Collar Value: {collar_price:,.2f}")
