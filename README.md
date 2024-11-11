import UIKit

class ViewController: UIViewController, UIPickerViewDelegate, UIPickerViewDataSource {
    
    @IBOutlet weak var amountTextField: UITextField!
    @IBOutlet weak var fromCurrencyTextField: UITextField!
    @IBOutlet weak var resultLabel: UILabel!
    @IBOutlet weak var currencyPicker: UIPickerView!
    
    let currencies = ["USD", "EUR", "GBP", "JPY", "AUD"] // List of currencies
    var exchangeRates: [String: Double] = [:] // Exchange rates will be populated via API
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        currencyPicker.delegate = self
        currencyPicker.dataSource = self
        
        fetchExchangeRates() // Fetch exchange rates when the app loads
    }
    
    // MARK: - UIPickerView Data Source
    
    func numberOfComponents(in pickerView: UIPickerView) -> Int {
        return 1
    }
    
    func pickerView(_ pickerView: UIPickerView, numberOfRowsInComponent component: Int) -> Int {
        return currencies.count
    }
    
    func pickerView(_ pickerView: UIPickerView, titleForRow row: Int, forComponent component: Int) -> String? {
        return currencies[row]
    }
    
    // MARK: - Button Action for Currency Conversion
    
    @IBAction func convertButtonTapped(_ sender: UIButton) {
        guard let amountText = amountTextField.text, let amount = Double(amountText),
              let fromCurrency = fromCurrencyTextField.text else { return }
        
        let toCurrency = currencies[currencyPicker.selectedRow(inComponent: 0)]
        
        // Perform currency conversion
        convertCurrency(amount: amount, fromCurrency: fromCurrency, toCurrency: toCurrency)
    }
    
    // MARK: - Fetch Exchange Rates
    
    func fetchExchangeRates() {
        let urlString = "https://api.exchangerate-api.com/v4/latest/USD" // You can use any exchange rate API
        guard let url = URL(string: urlString) else { return }
        
        URLSession.shared.dataTask(with: url) { data, response, error in
            if let error = error {
                print("Error fetching data: \(error)")
                return
            }
            
            guard let data = data else { return }
            
            do {
                // Parse the JSON data returned by the API
                let response = try JSONDecoder().decode(ExchangeRates.self, from: data)
                
                // Update exchange rates dictionary
                self.exchangeRates = response.rates
            } catch {
                print("Error decoding data: \(error)")
            }
        }.resume()
    }
    
    // MARK: - Convert Currency
    
    func convertCurrency(amount: Double, fromCurrency: String, toCurrency: String) {
        // Fetch the rates from the dictionary
        guard let fromRate = exchangeRates[fromCurrency],
              let toRate = exchangeRates[toCurrency] else {
            resultLabel.text = "Invalid currency"
            return
        }
        
        // Convert the amount from the base currency to the target currency
        let baseAmount = amount / fromRate
        let convertedAmount = baseAmount * toRate
        
        // Update the result label on the main thread
        DispatchQueue.main.async {
            self.resultLabel.text = String(format: "%.2f", convertedAmount)
        }
    }
}

import Foundation

// This model represents the response from the exchange rate API.
struct ExchangeRates: Decodable {
    let rates: [String: Double] // Dictionary of currency codes and their exchange rates
    let base: String // The base currency used for conversion (e.g., USD)
}

import Foundation

// This model represents the response from the exchange rate API.
struct ExchangeRates: Decodable {
    let rates: [String: Double] // Dictionary of currency codes and their exchange rates
    let base: String // The base currency used for conversion (e.g., USD)
}


