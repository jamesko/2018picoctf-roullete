# 2018picoctf-roullete
A solution to the 2018picoCTF roullete problem.

The source code for the roullete game revealed that placing large integer bets would give large negative values. If you lost the round during such a bet, your account would decreae by a negative amount -- meaning increase. Also, it seemed that there was some potential for the random number generator having an issue. I'm not a C programmer, and read that others had found this. The problem was that the random function was seeded using srand. Adn the seed was the cash value that the wallet/account you were given was populated with by the first use of the random number generator. Having the random func seed mean you can "predict" the stream of pseudo-random numbers on subsequent calls to the rand function.

My hacking solution was to not create an entirely new random function to simulate the bets, but to modify the existing source code of the game to populate the account with some integer value given in the argument of the command line call of the executable. Playing the live game in one console, and then launching the hacked version in the other with whatever the initial wallet is populate with.

Then either:
1. Win about 20 games by doubling the account each time.
2. Win 3 games by any amount, but lose the fouth using the bug that gives a very large negative bet.

</>
