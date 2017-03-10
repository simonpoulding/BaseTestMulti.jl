# BaseTestMulti

[![Build Status](https://travis-ci.org/simonpoulding/BaseTestMulti.jl.svg?branch=master)](https://travis-ci.org/simonpoulding/BaseTestMulti.jl)

[![Coverage Status](https://coveralls.io/repos/simonpoulding/BaseTestMulti.jl/badge.svg?branch=master&service=github)](https://coveralls.io/github/simonpoulding/BaseTestMulti.jl?branch=master)

[![codecov.io](http://codecov.io/github/simonpoulding/BaseTestMulti.jl/coverage.svg?branch=master)](http://codecov.io/github/simonpoulding/BaseTestMulti.jl?branch=master)


## Usage

To use BaseTestMulti features within particular a test set, specify it with `@mtestset` instead of `@testset`:

	using Base.Test
	using BaseTestMulti
	@mtestset "rand returns different values" begin
		x = rand(1:6)
		@test 1 <= x <= 6
		@mtest_values_vary x
	end

The test block is executed multiple times (30 times, by default).  `@test_...` macros are applied as normal: once for each iteration of the test block.  But `@mtest_...` macros collect the values being tested, and then apply a test to the collection of values once the all iterations are complete:

* `@mtest_values_vary x` tests that `x` takes at least two different values across the iterations
* `@mtest_values_are [3,2,1] x` tests that the unique values taken by `x` are 1, 2, and 3
* `@mtest_values_includes [1,2] x` tests that the set of values taken by `x` include 1 and 2
* `@mtest_that_sometimes y` tests that `y` is true at least one across the iterations
* `@mtest_distributed_as Uniform(1,6) z` tests the distribution of `z` that is not statistically significant different from the specified distribution.  The distribution can be any type to which the `rand(..., n::Int)` method can be applied, e.g. vectors, ranges, or -- as in this example -- univariate distributions from the Distributions package.

The following options are supported by `@mtestset`:

* `reps` - the number of iterations of the test block (default: 30)
* `alpha` - the significance level used by `@mtest_distributed_as` (default: 0.01)

For example:

	@mtestset "rand returns uniformally distributed values" reps=100 alpha=0.05 begin
		x = rand(1:6)
		@mtest_distributed_as Uniform(1,6) x
	end

To handle iterations within the test block itself, use `@testset MultiTestSet` instead of `@mtestset`:

	@testset MultiTestSet "rand returns uniformally distributed values" alpha=0.05 begin
		for i in 1:100
			x = rand(1:6)
			@mtest_distributed_as Uniform(1,6) x
		end
	end

`@mtestset` supports the same `for` syntax as `@testset`:

	@mtestset "rand returns uniformally distributed values across 1 to $bound" for bound in 6:20
		x = rand(1:bound)
		@mtest_distributed_as Uniform(1,bound) x
	end

(This is equivalent to 15 test sets -- one for each value of `bound` between 6 and 20 -- and the values collected by `@mtest...` macros do not accumulate across these test sets.) 

`@mtestset` can be embedded in a `@testset` and vice versa:

	@testset "my test set" begin
		@mtestset "rand returns different values" begin
			x = rand(1:6)
			@mtest_values_vary x
			@testset "superfluous" begin
				@test isa(x,Int)
			end
		end
	end

