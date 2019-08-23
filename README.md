# 2018picoctf-roullete
A solution to the 2018picoCTF roullete problem.

The source code for the roullete game revealed that placing excessively large integer bets would give large negative values. If you lost the round during such a bet, your account would decreae by a negative amount -- meaning increase. Also, it seemed that there was some potential that the random number generator having an issue. I'm not a C programmer, and researched how the urand function works. It was not clear how this would be exploited, though there did seem potential. However, I decided to read others findings and found that others had found that the srand (something I did not explore) which seeded the rand function. The problem was that the seed was the cash value that the game initialed your wallet/account with at the start of the game. Having the seed means you can "predict" (play out) the stream of pseudo-random numbers that would be retruned from the rand function on subsequent calls after seeding.

My hack solution was to not create an entirely new random function to simulate the bets, but to modify the existing source code of the game to accept whatever arg value was given at the command line at runtime as the account value and pass that on as the seed. Then, while playing the live game in one console, you could launch the hacked version in the other with whatever the initial wallet is populate with.

Then either:
1. Win about 20 games by doubling the account each time.
2. Win 3 games by any amount, but lose the fouth using the bug that gives a very large negative bet.

Details on source code modifications:
--------------------------------------
1. Give a parameter to get_rand function, and set the seed to this value.
``` C
    long get_rand(argSeed) {            <--- MOD
      long seed;
      FILE *f = fopen("/dev/urandom", "r");
      fread(&seed, sizeof(seed), 1, f);
      fclose(f);
      seed = argSeed; //seed % 5000;    <--- MOD
      if (seed < 0) seed = seed * -1;
      srand(seed);
      return seed;
    }
'''

2. Comment out one of the losing messages. The win scenario invoked the rand function once, and the lose scenario invoked it twice. This would desynccronize the hack game and live game because you would likely have losing guesses each round when attempting to discover the next random number. While, the live game would be all win scenarios because you are entering only correctly predicted values [other than the final round you might lose intentionally].

    void play_roulette(long choice, long bet) {

      printf("Spinning the Roulette for a chance to win $%lu!\n", 2*bet);
      long spin = (rand() % ROULETTE_SIZE)+1;

      spin_roulette(spin);

      if (spin == choice) {
        cash += 2*bet;
        puts(win_msgs[rand()%NUM_WIN_MSGS]);
        wins += 1;
      }
      else {
        puts(lose_msgs1[rand()%NUM_LOSE_MSGS]);
        //puts(lose_msgs2[rand()%NUM_LOSE_MSGS]);   <--- MOD
      }
      puts("");
    }

3. Take command line arguments as value for use in modified "get_rand" function to be used as the seed.

int main(int argc, char *argv[]) {
  setvbuf(stdout, NULL, _IONBF, 0);

  cash = get_rand(atoi(argv[1]));     <--- MOD

...
}


</>
