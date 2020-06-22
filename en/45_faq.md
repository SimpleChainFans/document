### Mine pool related issues

**1.What should I fill in the receiving address?**

We recommend that you fill in the SIPC wallet address, because you have the private key of the currency, which is more secure.

**2.What coins can SimPool dig ?**

Currently, SIPC mining is supported.

**3. How to mine ？**

The general steps are: register an account-create a mining account-configure a computer-view revenue. For specific steps, see SIPC mining tutorial.

**4. How high is my computing power?**

The actual computing power after connecting the mining pool for mining is not necessarily equal to the nominal computing power, and the specific computing power displayed by the mining pool is the standard.

**5. Why is my income decreasing?**

First confirm the following points:

- (1) whether the mining equipment operates normally;

- (2) whether the mining configuration is correct;

- (3) whether the network connection is normal;

- (4) whether the difficulty of mining changes (if the difficulty changes, the profit will fluctuate); And with the increase of mining equipment and the increase of the whole network computing power, the difficulty of mining will increase, the mining revenue of a single device will gradually decrease.

**5. When will the daily income be settled?**

The hourly settlement is conducted once every hour, and then the coin is made at 12 a.m. the specific calculation method of personal income (per hour) is as follows:

- Mining pool revenue = average computing power of the mining pool/computing power of the whole network * The number of SIPC output per day of the whole network + super-fast revenue;

- Personal income = total personal acceptance/total network acceptance * mine pool income;

**6. Why is the network unstable sometimes?**

Sometimes there will be technical adjustments, which are not stable, but generally can be solved in 30 to 2 hours. There may also be attacks, attacks against bitcoin networks, there are also attacks against mine pool servers. If it is a personal network problem, you need personal debugging to solve it or seek help from the network operator.

**7. What if the computing power drops?**

Please check the following questions first:

- (1) whether the mining equipment operates normally;

- (2) whether the mining configuration is correct;

- (3) whether the network connection is normal;

After the above troubleshooting, the computational power still decreases. Please find customer service in the Simpool group to solve the problem.

**8. What is workload? What is the network difficulty?**

The workload refers to the contribution of the miner to the computing power for digging new blocks. The network difficulty refers to the computing difficulty of the whole network explosion block, which is dynamically adjusted according to the total computing power of all miners.

**9. What are the settlement methods for the mine pool?**

At present, the mining pool adopts the PPLNS settlement method (full name: Pay Per Last N Shares), which means "Pay the income according to the past N Shares". This means that once all miners find a block, they will distribute the currency in the block according to the proportion of shares each person contributes.

**10. Will the revenue be lost if the collection address is not set?**

If you do not fill in the receiving address, your revenue will be temporarily stored in the Simpool account. After you set the address, we will pay you the income after the address takes effect.

**11. When using SimpleNode wallet to select mining pool for mining, how to set the parameters of advanced graphics card?**

Known video card tuning parameters:

- (1) `NVIDIA GTX-750-Ti`

      gpuplatform=2, memsize=1999, globalworksize=10240, localworksize=512, lookupgap=4

- (2) `AMD Ellesmere RX 570`

      gpuplatform=1, memsize=2048, globalworksize=8000, localworksize=64, lookupgap=2

- (3) `GeForce GT 1030`

      gpuplatform=2, memsize=2048, globalworksize=4096, localworksize=64, lookupgap=4

>`gpuplatform`: AMD graphics card fill in 1, NVIDIA graphics card fill in 2

>`memsize:` Filter graphics cards that are not less than the video memory capacity (default 2048MB), NVIDIA GTX-750-Ti video memory is 1999MB

>`globalworksize:` The parallel computing item is generally a multiple of 1024, and the maximum can not be greater than the Max work item sizes of the graphics card. However, when adjusting, AMD's RX570 setting 8000 has the highest efficiency.

>`localworksize:` The work item of each graphics card calculation unit is generally 64/128/256/512. globalworksize/localworksize cannot be greater than the Max work group size supported by the graphics card.

>`lookupgap:` This value is finally adjusted. The value range is 1/2/4/8. The default value is 2. Adjust the performance between video memory and video memory. When it is equal to 1, the memory used is the most. Based on this, 1/2 is used when it is equal to 2, and 1/8 is used when it is equal to 8.

### SimpleNode X1

**1.Can an account be associated with several more mining machines？**

Yes, as long as the power supply of the mining machine is guaranteed to be sufficient, multiple mining machines can be connected in series on one computer.

- If the Solo mining method is used, the profit will be directly transferred to the mining account address;
- If you use the mining pool mining method, the profit is transferred to the account address associated with the miner.

**2.How to check the status of each mining machine after connecting multiple mining machines?**

If multiple mining machines are mining at the same time, click the small arrow behind "stop mining" on SimpleNode software to view the status of each mining machine.

![7.18.png](https://i.loli.net/2020/05/07/H6j7wUXFPvMNu1R.png)

**3.How many mining machines can the network and computer at home drag at most?**

The network bandwidth has little impact on the mining machine, as long as the SimpleNode can be connected to the Simpool mining pool; As long as the power supply of the mining machine is guaranteed to be sufficient, each computer can be connected with multiple mining machines, generally, it can support about 2-3 sets at most; Additional external power supply is required.

