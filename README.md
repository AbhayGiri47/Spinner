# Spinner
Create Spinner in Android



<TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="@dimen/activity_horizontal_margin"
                        android:text="Quarters"
                        android:textColor="@color/colorAccent"
                        android:textSize="@dimen/textsize_14"
                        android:fontFamily="@font/montserrat_medium"/>

                <android.support.v7.widget.AppCompatSpinner
                        android:id="@+id/spinner_quarter"
                        android:layout_width="match_parent"
                        app:backgroundTint="@color/colorPrimary"
                        android:layout_height="@dimen/dimen_40dp"
                        android:layout_marginTop="@dimen/dimen_3dp"
                        android:textSize="@dimen/textsize_16"/>
                <View
                        android:layout_width="match_parent"
                        android:layout_height="@dimen/dimen_1dp"
                        android:background="@color/dark_gray" />

                <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="@dimen/activity_horizontal_margin"
                        android:text="Financial Year"
                        android:textColor="@color/colorAccent"
                        android:textSize="@dimen/textsize_14"
                        android:fontFamily="@font/montserrat_medium"/>

                <EditText
                        android:id="@+id/edt_year"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:hint="Year"
                        android:textSize="@dimen/textsize_17"
                        android:textColor="@color/text_color"
                        android:focusableInTouchMode="false"
                        android:drawableRight="@drawable/date_icon"
                        android:drawableEnd="@drawable/date_icon"/>

                <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="@dimen/dimen_30dp"
                        android:text="@string/report_category"
                        android:textColor="@color/colorAccent"
                        android:textSize="@dimen/textsize_14"
                        android:fontFamily="@font/montserrat_medium"/>

                <android.support.v7.widget.AppCompatSpinner
                        android:id="@+id/spinner_report_category"
                        android:layout_width="match_parent"
                        app:backgroundTint="@color/colorPrimary"
                        android:layout_height="@dimen/dimen_40dp"
                        android:layout_marginTop="@dimen/dimen_3dp"
                        android:textSize="@dimen/textsize_16"
                        android:overlapAnchor="false"/>

                <View
                        android:layout_width="match_parent"
                        android:layout_height="@dimen/dimen_1dp"
                        android:background="@color/dark_gray"/>
                        
                        
    private var ipcList = ArrayList<IPCData>()
    private var ipcNameLIst = ArrayList<String>()
    private var ipcIdList = ArrayList<Int>()
    private var ipcId = 0


    private var reportCategoryList = ArrayList<ReportCategoryData>()
    private var reportCategoryNameLIst = ArrayList<String>()
    private var reportCategoryIdList = ArrayList<Int>()
    private var reportCategoryId = 0
        private val periodList = arrayOf("Quarters", "Q1", "Q2", "Q3", "Q4")
        

private fun openMarketResearchDialog(reportType: String, marketResearchTypeId: Int) {

        val filterDialog = Dialog(activity)

        val filterBinding = DialogMarketResearchBinding.inflate(layoutInflater, null, false)

        filterDialog.setContentView(filterBinding.root)
        filterDialog.window?.setLayout(
            ViewGroup.LayoutParams.MATCH_PARENT,
            ViewGroup.LayoutParams.WRAP_CONTENT
        )
        filterYearAnalyst = filterBinding.edtYear

        val quarterAdapter =
            object : ArrayAdapter<String>(activity, R.layout.spinner_item, periodList) {
                override fun isEnabled(position: Int): Boolean {
                    return position != 0
                }

                override fun getDropDownView(
                    position: Int, convertView: View?,
                    parent: ViewGroup
                ): View {
                    val view = super.getDropDownView(position, convertView, parent)
                    val tv = view as TextView
                    if (position == 0) {
                        // Set the disable item text color
                        tv.setTextColor(Color.GRAY)
                    } else {
                        tv.setTextColor(Color.BLACK)
                    }
                    return view
                }
            }
        filterBinding.spinnerQuarter.adapter = quarterAdapter

        // Report Category
        viewModel.getReportCategoryList().observe(this, Observer {

            if (it != null) {
                if (it.code() == 200 && it.body() != null) {
                    if (it.body()!!.data != null && it.body()!!.data!!.isNotEmpty()) {

                        reportCategoryList.clear()
                        reportCategoryNameLIst.clear()
                        reportCategoryIdList.clear()

                        reportCategoryList.addAll(it.body()!!.data!!)

                        reportCategoryIdList.add(0)
                        reportCategoryNameLIst.add("Select Report Category")
                        for (i in reportCategoryList) {
                            reportCategoryNameLIst.add(i.name!!)
                            reportCategoryIdList.add(i.iD!!)
                        }

                        val reportCategoryAdapter = object : ArrayAdapter<String>(
                            activity, R.layout.spinner_item, reportCategoryNameLIst
                        ) {
                            override fun isEnabled(position: Int): Boolean {
                                return position != 0
                            }

                            override fun getDropDownView(
                                position: Int, convertView: View?,
                                parent: ViewGroup
                            ): View {
                                val view = super.getDropDownView(position, convertView, parent)
                                val tv = view as TextView
                                if (position == 0) {
                                    // Set the disable item text color
                                    tv.setTextColor(Color.GRAY)
                                } else {
                                    tv.setTextColor(Color.BLACK)
                                }
                                return view
                            }
                        }

                        filterBinding.spinnerReportCategory.adapter = reportCategoryAdapter

                        val pos = pref.get("reportCategory", 0)
                        if (reportCategoryNameLIst.size > pos) {
                            filterBinding.spinnerReportCategory.setSelection(pos)
                            val quarpos = when (pref.get("quarter", "")) {
                                "Quarter 1" -> 1
                                "Quarter 2" -> 2
                                "Quarter 3" -> 3
                                "Quarter 4" -> 4
                                else -> 0
                            }
                            filterBinding.spinnerQuarter.setSelection(quarpos)
                            filterBinding.edtYear.setText(pref.get("yearMarket", ""))
                        }
                    } else {
                        showToast("No data available")
                    }

                } else if (it.errorBody() != null) {
                    val data = it.errorBody()!!.string()
                    try {
                        val jObjError = JSONObject(data)
                        val msg = jObjError.getString("Message")
                        showToast(msg)
                    } catch (e: Exception) {
                        showToast(it.message())
                    }
                } else {
                    showToast(it.message())
                }
            } else {
                showToast("Something went wrong")
            }
        })


        // IPC
        viewModel.getIPCList().observe(this, Observer {
            if (it != null) {
                if (it.code() == 200 && it.body() != null) {
                    if (it.body()!!.data != null && it.body()!!.data!!.isNotEmpty()) {
                        ipcList.clear()
                        ipcIdList.clear()
                        ipcNameLIst.clear()

                        ipcList.addAll(it.body()!!.data!!)
                        ipcIdList.add(0)
                        ipcNameLIst.add("Select IPC")
                        for (i in ipcList) {
                            ipcNameLIst.add(i.name!!)
                            ipcIdList.add(i.iD!!)
                        }


                        val ipcAdapter = object : ArrayAdapter<String>(
                            activity, R.layout.spinner_item, ipcNameLIst
                        ) {
                            override fun isEnabled(position: Int): Boolean {
                                return position != 0
                            }

                            override fun getDropDownView(
                                position: Int, convertView: View?,
                                parent: ViewGroup
                            ): View {
                                val view = super.getDropDownView(position, convertView, parent)
                                val tv = view as TextView
                                if (position == 0) {
                                    // Set the disable item text color
                                    tv.setTextColor(Color.GRAY)
                                } else {
                                    tv.setTextColor(Color.BLACK)
                                }
                                return view
                            }
                        }

                        filterBinding.spinnerIpc.adapter = ipcAdapter
                        filterBinding.spinnerIpc.setSelection(pref.get("ipc", 0))
                    } else {
                        showToast("No data available")
                    }
                } else if (it.errorBody() != null) {
                    val data = it.errorBody()!!.string()
                    try {
                        val jObjError = JSONObject(data)
                        val msg = jObjError.getString("Message")
                        showToast(msg)
                    } catch (e: Exception) {
                        showToast(it.message())
                    }
                } else {
                    showToast(it.message())
                }
            } else {
                showToast("Something went wrong")
            }
        })

        viewModel.ipc(companyID)
        viewModel.reportCategory(companyID)


        filterBinding.spinnerQuarter.onItemSelectedListener =
            object : AdapterView.OnItemSelectedListener {
                override fun onNothingSelected(p0: AdapterView<*>?) {

                }

                override fun onItemSelected(
                    parent: AdapterView<*>?,
                    view: View?,
                    position: Int,
                    id: Long
                ) {
                    if (position > 0) {
                        //quarterPos = position
                        quarterPos = when (periodList[position]) {
                            "Q1" -> "Quarter 1"
                            "Q2" -> "Quarter 2"
                            "Q3" -> "Quarter 3"
                            "Q4" -> "Quarter 4"
                            else -> ""
                        }
                    }
                }

            }

        filterBinding.edtYear.setOnClickListener {
            /* dateFlag = 2
             CommonUtils.getCalenderDate(activity!!, this)*/
            val pickerDialog = MonthYearPickerDialog()
            pickerDialog.setListener { view, year, month, dayOfMonth ->
                this.yearAnalyst = year
                filterBinding.edtYear.setText(year.toString())
            }
            pickerDialog.show(fragmentManager, "MonthYearPickerDialog")
        }

        filterBinding.spinnerReportCategory.onItemSelectedListener =
            object : AdapterView.OnItemSelectedListener {
                override fun onNothingSelected(parent: AdapterView<*>?) {
                }

                override fun onItemSelected(
                    parent: AdapterView<*>?,
                    view: View?,
                    position: Int,
                    id: Long
                ) {
                    if (position > 0)
                        reportCategorypos = position
                    reportCategoryId = reportCategoryIdList[position]
                }
            }

        filterBinding.spinnerIpc.onItemSelectedListener =
            object : AdapterView.OnItemSelectedListener {
                override fun onNothingSelected(parent: AdapterView<*>?) {
                }

                override fun onItemSelected(
                    parent: AdapterView<*>?,
                    view: View?,
                    position: Int,
                    id: Long
                ) {

                    if (position > 0) {
                        ipcpos = position
                        ipcId = ipcIdList[position]
                        //  viewModel.ipc(companyID)
                    }
                }
            }


        filterBinding.btnApplyfilter.setOnClickListener {

            if (validate(quarterPos, filterBinding.edtYear.text.toString().trim())) {
                pref["reportCategory"] = reportCategorypos
                pref["ipc"] = ipcpos
                pref["type"] = 0
                pref.set("yearMarket", filterBinding.edtYear.text.toString().trim())
                pref.set("quarter", quarterPos)

                filterDialog.dismiss()
            }


        }

        filterBinding.txtReset.setOnClickListener {
            filterBinding.spinnerReportCategory.setSelection(0)
            filterBinding.spinnerQuarter.setSelection(0)
            filterBinding.spinnerIpc.setSelection(0)
            filterBinding.edtYear.text.clear()

            reportCategorypos = 0
            ipcpos = 0
            quarterPos = ""
            yearAnalyst = 0

            pref["reportCategory"] = 0
            pref["ipc"] = 0
            pref["quarter"] = ""
            pref["yearMarket"] = ""

        }

        filterDialog.show()
    }

    fun validate(quarterPos: String, year: String): Boolean {
        var temp = true
        if (quarterPos.isNotEmpty() && year.isNotEmpty()) {
            temp = true
        } else if (quarterPos.isEmpty() && year.isNotEmpty()) {
            showToast("Please select quarter if year is selected")
            temp = false
        } else if (quarterPos.isNotEmpty() && year.isEmpty()) {
            showToast("Please select year if quarter is selected")
            temp = false
        }
        return temp
    }


    override fun onDatePick(date: String) {
        val dateInString = DateParser.parseDateToYearFirst(
            date,
            Constant.SYSTEM_DATE_FORMAT,
            Constant.DATE_FORMAT_YEAR
        )

        selectedStartDate = dateInString
        asOnDate!!.setText(selectedStartDate)

        if (dateFlag == 1) {
            /* val splitApiDate = date.split("/")
             month = splitApiDate[0].toInt()

             val formattedDate = sdfInput.parse(date)

             val formattedDateInString = sdfOutput.format(formattedDate)
             val splitDate = formattedDateInString.split("/")

             monthString = splitDate[0]
             val date: Int = splitDate[1].toInt()
             val year: Int = splitDate[2].toInt()

             filterMonth?.setText(monthString)*/

        } else {
            val splitDate = date.split("/")
            val month: Int = splitDate[0].toInt()
            val date: Int = splitDate[1].toInt()
            yearAnalyst = splitDate[2].toInt()
            filterYearAnalyst?.setText(yearAnalyst.toString())
        }

    }
