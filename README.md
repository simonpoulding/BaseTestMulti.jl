# BaseTestMulti

[![Build Status](https://travis-ci.org/simonpoulding/BaseTestMulti.jl.svg?branch=master)](https://travis-ci.org/simonpoulding/BaseTestMulti.jl)

[![Coverage Status](https://coveralls.io/repos/simonpoulding/BaseTestMulti.jl/badge.svg?branch=master&service=github)](https://coveralls.io/github/simonpoulding/BaseTestMulti.jl?branch=master)

[![codecov.io](http://codecov.io/github/simonpoulding/BaseTestMulti.jl/coverage.svg?branch=master)](http://codecov.io/github/simonpoulding/BaseTestMulti.jl?branch=master)


BaseTestMulti extends Base.Test to facilitate tests over multiple values.

`@mtestset "example of autorepeating (defaults to 30 repetitions)" begin
	@mtest_values_vary rand(1:6)
end`

@mtestset "example of autorepeating - set explicitly" reps=7 begin
	@mtest_values_vary rand(1:6)
end

@mtestset "can use for syntax in same way as for @testset, and similarly creates a new test set for each value of loop var" for i in 1:10
	@mtest_values_vary rand(1:i)
end

using Distributions
@mtestset "compares to a distribution (any type that rand(...) accepts) using MannWhitneyU with defined significance" begin
	x = rand(1:6)
	@mtest_distributed_as x DiscreteUniform(1,6) 0.01
	@mtest_distributed_as x 1:6 0.01
end

@testset MultiTestSet "omit automatic looping" begin
	for x in ["i","j","k"]
		@mtest_values_include x ["k","i"]
	end
end

