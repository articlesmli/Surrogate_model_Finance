### A surrogate model (a simplified, fast model) for financial option pricing

The logic behind using neural networks to price American options is a well-established concept in academic and professional quantitative finance. However, the specific implementation in this notebook does address several practical gaps often found in textbook theory versus real-world application, such as: 
- The "Speed-Accuracy" Gap: Traditional models like the Binomial Tree or Monte Carlo simulations are highly accurate but computationally expensive, making them difficult to use for real-time risk management of large portfolios.
- Generalisation vs. Point Calculation: There is often a disconnect between calculating a single price and understanding how that price moves across a broad "map" of varying market conditions
- Early Exercise Complexity: The pricing of American options is significantly more complex than European options because they can be exercised at any time. Standard Black-Scholes formulas (common in introductory literature) cannot price American options. While the Binomial Tree can, it becomes "heavier" as the number of steps (N) increases.

The proposed methodolofy effectively fills these practical gaps in the literature by providing a scalable solution for real-time risk management and large-scale portfolio valuation, where traditional recursive models are too slow for high-frequency environments. It specifically uses a Neural Network to mimic a slower, traditional mathematical model called a Binomial Tree. Here is a breakdown of what the code is doing step-by-step:
1. Creating the "Ground Truth" (Binomial Tree). The function binomial_tree_american_option represents a high-fidelity model that calculates the price of an American Put Option. It uses backward induction through a price tree to determine the option's value today, accounting for the possibility of early exercise. This is the "correct" but computationally expensive way to price these options. However, this "expensive" code is the investment, and the later "Surrogate Model" (the Neural Network) is the profit. i.e. it is spent computational power upfront to buy infinite speed later.

Here is why this workflow is valuable in a professional finance setting:
- The "Train Once, Predict Millions" Advantage. The expensive Binomial Tree only needs to run 1,000 times to create the training data. This might take a few seconds or minutes. However, once the Neural Network is trained, it can perform millions of calculations per second. Running the "What-If" scenarios across an entire portfolio of thousands of options instantly, which would be impossible using the Tree model alone.
- Eliminating the "Curse of Dimensionality". The increase of the number of steps (N) in a Binomial Tree to get better accuracy leads to growing of the computation time exponentially; however, the Neural Network’s speed is independent of how complex the original model was. Whether the "Teacher" model took 1 second or 1 hour to calculate a price, the NN takes the same micro-fraction of a second to mimic the answer.
- Creating a Continuous Pricing Surface. The Binomial Tree gives a single answer for a single set of inputs. It is "discrete." The Neural Network creates a smooth, mathematical function (a pricing surface). This allows us to calculate Greeks (Delta, Gamma, Vega) much more easily through differentiation rather than rebuilding trees for every tiny price movement.
- Portability and Integration. A Binomial Tree requires a lot of logic, loops, and memory management to run. A trained Neural Network is just a set of weights. You can export those weights and put them into a mobile app, a lightweight web browser, or a high-frequency trading execution system where you don't have the space or time to run a full Binomial simulation.

2. Generating a Training Dataset. The generate_dataset function creates 1,000 random market scenarios. It varies four key inputs:
S - Stock Price
T - Time to Expiryr
r - Risk-free interest ratesigma
sigma - Volatility.

For each scenario, it runs the slow Binomial Tree model to find the "True" price. This creates a "map" of market conditions and their resulting prices for the AI to study.

3. Building and Training the Surrogate. The code then builds a Deep Neural Network (specifically an MLPRegressor with three hidden layers).
   - Scaling: It uses StandardScaler to normalize the data, which is critical for helping Neural Networks learn efficiently.
   - Learning: The model "learns" the complex mathematical relationship between the four market inputs and the final option price.

4. Evaluating Results and SpeedFinally. Finally, the code tests the surrogate's performance on data it hasn't seen before.
   - Accuracy: In the output shown, the model achieved an R^2 score of 0.9996, meaning it is nearly 100% accurate at mimicking the slow model.
   - Speed: The surrogate is exponentially faster, making predictions in roughly 0.000005 seconds per calculation.

5. Sensitivity Analysis (Visualization). The last part of the code creates plots to see how the predicted option price changes as one variable (like Stock Price or Volatility) moves while the others stay constant. This helps verify that the AI has learned the correct financial logic (e.g., as volatility increases, the option price should go up).
