<h3>An array's length should be cached to save gas in for-loops</h3>
<h5> Link to code: https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol</h5>

<h5>I suggest storing the array`s length in a variable before the for-loop and use it instead: 

<h5>src/vault/PermissionRegistry.sol:42  for (uint256 i = 0; i < len; i++)</h5>

<h3>++i costs less gas compared to i++</h3>

<h5>i++ increments i and returns the initial value of i. Which means,for example:</h5>

<p>uint i = 1;<br>
i++; // == 1 but i == 2 </p>

<h5>However, ++i returns the actual incremented value:</h5>

<p>uint i = 1;<br>
++i; //  == 2 and i == 2 too, so there is no need for a temporary variable</p>

<h6>When we take the first case ( i++),the compiler has to create a temporary variable</h6>



